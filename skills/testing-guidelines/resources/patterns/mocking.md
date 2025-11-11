# Testing Patterns - Mocking and Async

Patterns for mocking dependencies, testing async functions, and handling errors. Part of the [Testing Guidelines](../../SKILL.md) skill.

## Table of Contents

1. [Mocking Dependencies](#mocking-dependencies)
2. [Testing Async Functions](#testing-async-functions)
3. [Testing Error Cases](#testing-error-cases)
4. [Mocking Environment Variables](#mocking-environment-variables)

---

## Mocking Dependencies

### Basic Mock Setup

```typescript
import { vi } from 'vitest';

const mockRedis = {
    setex: vi.fn(),
    get: vi.fn(),
    del: vi.fn(),
};
```

### Mocking Module Imports

```typescript
import { vi } from 'vitest';

// Mock entire module
vi.mock('./database', () => ({
    Database: vi.fn(() => ({
        query: vi.fn(),
        connect: vi.fn(),
    })),
}));

// Mock specific functions
vi.mock('./utils', () => ({
    calculateTax: vi.fn(),
    formatCurrency: vi.fn(),
}));
```

### Mocking Return Values

```typescript
// Simple return value
mockService.getData.mockReturnValue({ id: '123', data: 'test' });

// Different returns for multiple calls
mockService.getData
    .mockReturnValueOnce('first')
    .mockReturnValueOnce('second')
    .mockReturnValue('default');

// Async returns
mockService.fetchData.mockResolvedValue({ success: true });
mockService.fetchData.mockRejectedValue(new Error('Network error'));
```

### Spying on Functions

```typescript
const spy = vi.spyOn(service, 'methodName');

// Spy without changing implementation
spy.mockImplementation(() => 'mocked value');

// Check if called
expect(spy).toHaveBeenCalled();
expect(spy).toHaveBeenCalledWith('arg1', 'arg2');
expect(spy).toHaveBeenCalledTimes(2);

// Restore original implementation
spy.mockRestore();
```

### Best Practice: Always Use toHaveBeenCalledWith

**Principle:** Use `toHaveBeenCalledWith()` instead of just `toHaveBeenCalled()` for precise verification.

```typescript
// ❌ Bad: Only checks if function was called
expect(detectAndReportSpy).toHaveBeenCalled();

// ✅ Good: Verifies exact arguments passed
expect(detectAndReportSpy).toHaveBeenCalledWith('deploy-stale');

// ✅ Also good: Use matchers for flexible verification
expect(mockService.save).toHaveBeenCalledWith({
    id: expect.any(String),
    timestamp: expect.any(Number),
    status: 'completed'
});

// ✅ For complex objects, use expect.objectContaining
expect(mockLogger.info).toHaveBeenCalledWith(
    expect.objectContaining({
        event: 'user_created',
        userId: 'user-123'
    })
);
```

**Why this matters:**
- Catches bugs where function is called with wrong arguments
- Documents expected behavior clearly
- Prevents false positives (function called but with incorrect data)
- Makes test failures more informative

**Exception:** Only use `toHaveBeenCalled()` when you truly don't care about arguments (rare).

---

## Testing Async Functions

### Basic Async Test

```typescript
it('should handle async operations', async () => {
    const result = await service.fetchData();
    expect(result).toEqual({ id: '123', data: 'test' });
});
```

### Testing Promises

```typescript
it('should resolve with correct data', async () => {
    await expect(service.getData()).resolves.toEqual({ success: true });
});

it('should reject with error', async () => {
    await expect(service.getData()).rejects.toThrow('Not found');
});
```

### Testing Promise.all

```typescript
it('should fetch multiple resources in parallel', async () => {
    mockApi.get
        .mockResolvedValueOnce({ data: 'resource1' })
        .mockResolvedValueOnce({ data: 'resource2' });

    const results = await service.fetchMultiple(['id1', 'id2']);

    expect(results).toEqual([
        { data: 'resource1' },
        { data: 'resource2' }
    ]);
});
```

### Testing with Delays

**Important:** Always use `afterEach` for timer cleanup, even for single tests. This ensures cleanup runs even when tests throw.

```typescript
describe('TimeoutService', () => {
    afterEach(() => {
        vi.useRealTimers(); // Cleanup runs even if test throws
    });

    it('should timeout after 5 seconds', async () => {
        vi.useFakeTimers();

        const promise = service.fetchWithTimeout(5000);

        // Fast-forward time
        vi.advanceTimersByTime(5000);

        await expect(promise).rejects.toThrow('Timeout');
    });
});
```

---

## Testing Error Cases

### Synchronous Errors

```typescript
it('should throw error when id is missing', () => {
    expect(() => service.process(null)).toThrow('ID is required');
});

it('should throw specific error type', () => {
    expect(() => service.process(null)).toThrow(ValidationError);
});

it('should throw error with message', () => {
    expect(() => service.process(null)).toThrow(/ID is required/);
});
```

### Async Errors

```typescript
it('should reject when validation fails', async () => {
    await expect(service.validate(invalid)).rejects.toThrow('Invalid input');
});

it('should handle network errors', async () => {
    mockApi.get.mockRejectedValue(new Error('Network error'));

    await expect(service.fetchData()).rejects.toThrow('Network error');
});
```

### Best Practice: Always Test Exception Messages

**Principle:** Don't just test that an exception was thrown - verify the exact message for better debugging.

```typescript
// ❌ Bad: Only tests that an exception was thrown
await expect(
    controller.handleCompletion(payload, 'succeeded')
).rejects.toThrow(BadRequestException);

// ✅ Good: Tests the specific exception message
await expect(
    controller.handleCompletion(payload, 'succeeded')
).rejects.toThrow('Deployment metadata not found');

// ✅ Also good: Test both exception type and message
await expect(
    controller.handleCompletion(payload, 'succeeded')
).rejects.toThrow(
    expect.objectContaining({
        message: 'Deployment metadata not found',
        statusCode: 400
    })
);
```

**Why this matters:**
- Specific error messages help you understand what failed
- Prevents false positives (wrong error thrown but test passes)
- Makes tests more maintainable when error messages change
- Better debugging when tests fail

### Error Handling

```typescript
it('should catch and log errors', async () => {
    mockService.process.mockRejectedValue(new Error('Processing failed'));

    await service.handleRequest(data);

    expect(mockLogger.error).toHaveBeenCalledWith(
        expect.stringContaining('Processing failed')
    );
});

it('should return default value on error', async () => {
    mockService.getData.mockRejectedValue(new Error('Not found'));

    const result = await service.getOrDefault('key', 'default');

    expect(result).toBe('default');
});
```

---

## Mocking Environment Variables

### Always Use vi.stubEnv for Environment Variables

**Principle:** Never directly modify `process.env` in tests. Use `vi.stubEnv()` and ALWAYS clean up with `vi.unstubAllEnvs()` in `afterEach`.

**Why:**
- Direct `process.env` mutation can leak between tests
- `afterEach` cleanup runs even when tests fail or throw
- More explicit and testable
- Prevents flaky tests from environment pollution

```typescript
// ❌ Bad: Direct process.env mutation
describe('ConfigService', () => {
    it('should use production config', () => {
        process.env.NODE_ENV = 'production';

        const config = loadConfig();

        expect(config.debug).toBe(false);
        // Problem: Environment leaks to other tests if this test fails
    });
});

// ✅ Good: Use vi.stubEnv with cleanup in afterEach (even for single test)
describe('ConfigService', () => {
    afterEach(() => {
        vi.unstubAllEnvs(); // Runs even if test throws
    });

    it('should use production config', () => {
        vi.stubEnv('NODE_ENV', 'production');

        const config = loadConfig();

        expect(config.debug).toBe(false);
    });
});
```

### Pattern: Always Use afterEach

```typescript
describe('ConfigService', () => {
    afterEach(() => {
        vi.unstubAllEnvs();
    });

    it('should load development config', () => {
        vi.stubEnv('NODE_ENV', 'development');
        vi.stubEnv('DEBUG', 'true');

        const config = service.loadConfig();

        expect(config.debug).toBe(true);
        expect(config.logLevel).toBe('debug');
    });

    it('should load production config', () => {
        vi.stubEnv('NODE_ENV', 'production');
        vi.stubEnv('DEBUG', 'false');

        const config = service.loadConfig();

        expect(config.debug).toBe(false);
        expect(config.logLevel).toBe('error');
    });
});
```

### Pattern: Testing Missing Environment Variables

```typescript
describe('ConfigService', () => {
    afterEach(() => {
        vi.unstubAllEnvs();
    });

    it('should throw when required env var is missing', () => {
        vi.stubEnv('API_KEY', undefined);

        expect(() => service.initialize()).toThrow('API_KEY is required');
    });

    it('should use default when optional env var is missing', () => {
        vi.stubEnv('PORT', undefined);

        const config = service.loadConfig();

        expect(config.port).toBe(3000); // Default value
    });
});
```

### Pattern: Testing Multiple Environment Scenarios

```typescript
describe('DatabaseService', () => {
    afterEach(() => {
        vi.unstubAllEnvs();
    });

    describe('in development', () => {
        beforeEach(() => {
            vi.stubEnv('NODE_ENV', 'development');
            vi.stubEnv('DB_HOST', 'localhost');
            vi.stubEnv('DB_PORT', '5432');
        });

        it('should connect to local database', async () => {
            await service.connect();

            expect(mockDb.connect).toHaveBeenCalledWith({
                host: 'localhost',
                port: 5432,
            });
        });

        it('should enable query logging', async () => {
            await service.connect();

            expect(service.isQueryLoggingEnabled()).toBe(true);
        });
    });

    describe('in production', () => {
        beforeEach(() => {
            vi.stubEnv('NODE_ENV', 'production');
            vi.stubEnv('DB_HOST', 'db.example.com');
            vi.stubEnv('DB_PORT', '5432');
        });

        it('should connect to production database', async () => {
            await service.connect();

            expect(mockDb.connect).toHaveBeenCalledWith({
                host: 'db.example.com',
                port: 5432,
            });
        });

        it('should disable query logging', async () => {
            await service.connect();

            expect(service.isQueryLoggingEnabled()).toBe(false);
        });
    });
});
```

### Guidelines

1. **Always use `vi.stubEnv()`** - Never mutate `process.env` directly
2. **ALWAYS clean up in `afterEach`** - Even for single tests (cleanup runs if test throws)
3. **Never inline cleanup** - Don't call `vi.unstubAllEnvs()` at end of test
4. **Group by environment** - Use nested `describe` blocks for different env scenarios
5. **Test undefined** - Explicitly test behavior when env vars are missing
6. **Stub all env vars** - Don't rely on real environment values in tests

### Benefits

✅ **No test pollution** - Environment changes don't leak between tests
✅ **Explicit dependencies** - Clear which env vars each test needs
✅ **Guaranteed cleanup** - `afterEach` runs even when tests throw or fail
✅ **Better isolation** - Tests don't depend on machine environment
✅ **Easier debugging** - Know exactly what environment each test uses

---

