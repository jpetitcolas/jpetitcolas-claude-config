# Testing Patterns - Logs and Observability

Patterns for testing logs, log levels, structured logging, and observability. Part of the [Testing Guidelines](../../SKILL.md) skill.

## Table of Contents

1. [Logs Are Part of Your System's Behavior](#logs-are-part-of-your-systems-behavior)
2. [Testing Log Levels and Properties](#testing-log-levels-and-properties)
3. [Testing Structured Logs](#testing-structured-logs)
4. [Testing Log Levels Matter](#testing-log-levels-matter)
5. [Testing Required Log Properties](#testing-required-log-properties)
6. [Testing Sensitive Data Redaction](#testing-sensitive-data-redaction)
7. [Verify Logs Weren't Called](#pattern-verify-logs-werent-called)

---

## Testing Logs and Observability

### Logs Are Part of Your System's Behavior

**Principle:** Logs provide observability into your system. Test them like any other behavior, including log levels and important properties.

**Why:**
- **Observability matters** - Logs are how you debug production issues
- **Log levels matter** - warn (40) vs error (50) affects alerting
- **Log structure matters** - Missing properties break log aggregation
- **Prevents regressions** - Ensure critical logs aren't accidentally removed

### Testing Log Levels and Properties

```typescript
describe('StaleEventDetector', () => {
    let controller: StaleEventDetector;
    let mockLogger: Logger;

    beforeEach(() => {
        mockLogger = {
            info: vi.fn(),
            warn: vi.fn(),
            error: vi.fn(),
            debug: vi.fn(),
        };
        controller = new StaleEventDetector(mockLogger);
    });

    it('should log warning with correct level and properties when detection fails', async () => {
        const error = new Error('Redis connection failed');
        vi.mocked(mockTrackingService.findStale).mockRejectedValue(error);

        await controller.detectAndReportStaleEvents();

        expect(mockLogger.warn).toHaveBeenCalledWith({
            event: 'stale_detection_failed',
            message: 'Failed to detect stale deployments',
            error: 'Redis connection failed',
        });
    });

    it('should log error with correct level when processing fails', async () => {
        vi.mocked(mockDatadogService.sendMetric).mockRejectedValue(
            new Error('Network error')
        );

        await controller.processEvent(event);

        expect(mockLogger.error).toHaveBeenCalledWith(
            expect.objectContaining({
                event: 'processing_failed',
                error: 'Network error',
            })
        );
    });
});
```

### Testing Structured Logs

When using structured logging (JSON logs), test the structure:

```typescript
describe('UserService', () => {
    let mockLogger: Logger;

    beforeEach(() => {
        mockLogger = {
            info: vi.fn(),
            warn: vi.fn(),
            error: vi.fn(),
        };
    });

    it('should log user creation with all required fields', async () => {
        const userData = {
            email: 'alice@example.com',
            name: 'Alice',
        };

        await service.createUser(userData, mockLogger);

        expect(mockLogger.info).toHaveBeenCalledWith({
            event: 'user_created',
            user_id: expect.any(String),
            email: 'alice@example.com',
            timestamp: expect.any(Number),
        });
    });

    it('should log validation errors with details', async () => {
        const invalidData = { email: 'invalid' };

        await expect(service.createUser(invalidData, mockLogger)).rejects.toThrow();

        expect(mockLogger.warn).toHaveBeenCalledWith({
            event: 'validation_failed',
            field: 'email',
            value: 'invalid',
            reason: 'Invalid email format',
        });
    });
});
```

### Testing Log Levels Matter

Different log levels have different operational meanings:

```typescript
describe('PaymentService', () => {
    it('should use INFO level for successful payment', async () => {
        await service.processPayment(validPayment);

        // ✅ Info level (30) - normal operation
        expect(mockLogger.info).toHaveBeenCalledWith(
            expect.objectContaining({
                event: 'payment_processed',
                amount: 100,
                currency: 'USD',
            })
        );
    });

    it('should use WARN level for retryable failures', async () => {
        mockPaymentGateway.charge.mockRejectedValueOnce(new Error('Timeout'));

        await service.processPayment(validPayment);

        // ✅ Warn level (40) - retryable issue
        expect(mockLogger.warn).toHaveBeenCalledWith(
            expect.objectContaining({
                event: 'payment_retry',
                error: 'Timeout',
                attempt: 1,
            })
        );
    });

    it('should use ERROR level for permanent failures', async () => {
        mockPaymentGateway.charge.mockRejectedValue(new Error('Card declined'));

        await expect(service.processPayment(validPayment)).rejects.toThrow();

        // ✅ Error level (50) - permanent failure
        expect(mockLogger.error).toHaveBeenCalledWith(
            expect.objectContaining({
                event: 'payment_failed',
                error: 'Card declined',
                user_id: 'user-123',
            })
        );
    });
});
```

### Testing Required Log Properties

Ensure logs contain properties needed for debugging and monitoring:

```typescript
describe('APIController', () => {
    it('should log request with correlation ID', async () => {
        const request = {
            method: 'POST',
            path: '/api/users',
            headers: { 'x-correlation-id': 'req-abc-123' },
        };

        await controller.handleRequest(request);

        expect(mockLogger.info).toHaveBeenCalledWith(
            expect.objectContaining({
                correlation_id: 'req-abc-123',
            })
        );
    });

    describe('environment-specific logging', () => {
        afterEach(() => {
            vi.unstubAllEnvs();
        });

        it('should log errors with stack trace in development', async () => {
            vi.stubEnv('NODE_ENV', 'development');
            const error = new Error('Database error');

            await controller.handleError(error);

            expect(mockLogger.error).toHaveBeenCalledWith(
                expect.objectContaining({
                    event: 'request_error',
                    error: 'Database error',
                    stack: expect.stringContaining('Error: Database error'),
                })
            );
        });

        it('should NOT log stack trace in production', async () => {
            vi.stubEnv('NODE_ENV', 'production');
            const error = new Error('Database error');

            await controller.handleError(error);

            const logCall = mockLogger.error.mock.calls[0][0];
            expect(logCall).not.toHaveProperty('stack');
            expect(logCall.error).toBe('Database error');
        });
    });
});
```

### Testing Sensitive Data Redaction

Ensure logs don't leak sensitive information:

```typescript
describe('AuthService', () => {
    it('should NOT log passwords in authentication attempts', async () => {
        await service.login({
            username: 'alice',
            password: 'secret123',
        });

        // ✅ Ensure password is NOT in logs
        const logCalls = mockLogger.info.mock.calls;
        logCalls.forEach(call => {
            const logData = JSON.stringify(call);
            expect(logData).not.toContain('secret123');
            expect(logData).not.toContain('password');
        });
    });

    it('should log redacted credit card numbers', async () => {
        await service.processPayment({
            cardNumber: '4532123456789012',
            amount: 100,
        });

        expect(mockLogger.info).toHaveBeenCalledWith(
            expect.objectContaining({
                event: 'payment_initiated',
                card_last_four: '9012',
                amount: 100,
            })
        );

        // ✅ Full card number must NOT appear
        const logCalls = mockLogger.info.mock.calls;
        logCalls.forEach(call => {
            expect(JSON.stringify(call)).not.toContain('4532123456789012');
        });
    });
});
```

### Pattern: Verify Logs Weren't Called

Sometimes it's important to verify logs did NOT occur:

```typescript
describe('CacheService', () => {
    it('should NOT log on cache hit', async () => {
        mockCache.get.mockResolvedValue({ data: 'cached' });

        await service.getData('key-123');

        // ✅ No logs for cache hits (too noisy)
        expect(mockLogger.info).not.toHaveBeenCalled();
        expect(mockLogger.debug).not.toHaveBeenCalled();
    });

    it('should log on cache miss', async () => {
        mockCache.get.mockResolvedValue(null);

        await service.getData('key-123');

        expect(mockLogger.debug).toHaveBeenCalledWith(
            expect.objectContaining({
                event: 'cache_miss',
                key: 'key-123',
            })
        );
    });
});
```

### Guidelines

1. **Test log levels** - info vs warn vs error matters operationally
2. **Test log structure** - Ensure all required fields present
3. **Test log content** - Verify important properties and values
4. **Test redaction** - Sensitive data must not appear in logs
5. **Test absence** - Verify logs NOT called when inappropriate
6. **Use `expect.objectContaining`** - For flexible log property matching
7. **Test by log level** - Separate tests for different severity levels

### Benefits

✅ **Catch missing logs** - Ensure observability isn't broken
✅ **Catch log regressions** - Properties accidentally removed
✅ **Verify log levels** - Correct severity for alerting
✅ **Ensure security** - Sensitive data not leaked
✅ **Documentation** - Tests show what logs are expected

---

