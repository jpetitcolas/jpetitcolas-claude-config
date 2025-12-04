# Testing Patterns - Time Mocking

Patterns for mocking time in tests to ensure deterministic behavior.

## Table of Contents

1. [Why Mock Time](#why-mock-time)
2. [Basic Pattern](#basic-pattern)
3. [Testing Timestamp Values](#testing-timestamp-values)
4. [Testing Time-Based Logic](#testing-time-based-logic)
5. [Guidelines](#guidelines)

---

## Why Mock Time

**Problem:** Tests using `new Date()` are non-deterministic and fail randomly.

**Solution:** Always mock time with `vi.useFakeTimers()` and `vi.setSystemTime()`.

**Benefits:**
- **Deterministic tests** - Same result every time
- **No flaky tests** - Time doesn't drift between assertion and action
- **Testable edge cases** - Easily test specific dates, timezones, boundaries

---

## Basic Pattern

```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';

describe('MyService', () => {
    beforeEach(() => {
        vi.useFakeTimers();
        vi.setSystemTime(new Date('2025-05-05 12:00:00'));
    });

    afterEach(() => {
        vi.useRealTimers();
    });

    it('should use mocked time', () => {
        const result = service.getCurrentTimestamp();
        expect(result).toBe(1746446400000); // Deterministic!
    });
});
```

---

## Testing Timestamp Values

### Don't: Use `new Date()` in Assertions

```typescript
// ❌ DON'T: Use new Date() in assertions
it('should record current timestamp', () => {
    const result = service.getCurrentTimestamp();
    expect(result).toBe(new Date().getTime()); // Flaky! Fails randomly
});

// ❌ DON'T: Compare against current time with ranges
it('should create user with timestamp', async () => {
    const user = await service.createUser({ email: 'test@example.com' });
    expect(user.createdAt).toBeGreaterThan(Date.now() - 1000); // Unreliable
});
```

### Do: Mock Time and Use Hard-Coded Values

```typescript
// ✅ DO: Mock time and use hard-coded expected values
beforeEach(() => {
    vi.useFakeTimers();
    vi.setSystemTime(new Date('2025-05-05 12:00:00'));
});

afterEach(() => {
    vi.useRealTimers();
});

it('should record timestamp at fixed time', () => {
    const result = service.getCurrentTimestamp();
    expect(result).toBe(1746446400000); // Deterministic
});

it('should create user with mocked timestamp', async () => {
    const user = await service.createUser({ email: 'test@example.com' });
    expect(user.createdAt).toBe(1746446400000); // Exact value
});
```

---

## Testing Time-Based Logic

### Testing Expiration

```typescript
describe('TokenService', () => {
    const FIXED_TIME = new Date('2025-01-15T10:00:00Z').getTime();

    beforeEach(() => {
        vi.useFakeTimers();
        vi.setSystemTime(FIXED_TIME);
    });

    afterEach(() => {
        vi.useRealTimers();
    });

    it('should create token with 1-hour expiration', () => {
        const token = service.createToken();

        expect(token.expiresAt).toBe(FIXED_TIME + 3600000); // +1 hour
    });

    it('should detect expired token', () => {
        const token = { expiresAt: FIXED_TIME - 1000 }; // Expired 1 second ago

        expect(service.isExpired(token)).toBe(true);
    });

    it('should detect valid token', () => {
        const token = { expiresAt: FIXED_TIME + 1000 }; // Expires in 1 second

        expect(service.isExpired(token)).toBe(false);
    });
});
```

### Testing Time Advancement

```typescript
describe('CacheService', () => {
    beforeEach(() => {
        vi.useFakeTimers();
        vi.setSystemTime(new Date('2025-01-15T10:00:00Z'));
    });

    afterEach(() => {
        vi.useRealTimers();
    });

    it('should expire cache after TTL', async () => {
        await service.set('key', 'value', { ttlMs: 5000 });

        // Advance time by 4 seconds - should still be cached
        vi.advanceTimersByTime(4000);
        expect(await service.get('key')).toBe('value');

        // Advance time by 2 more seconds - should be expired
        vi.advanceTimersByTime(2000);
        expect(await service.get('key')).toBeNull();
    });
});
```

### Testing Date Boundaries

```typescript
describe('ReportService', () => {
    afterEach(() => {
        vi.useRealTimers();
    });

    it('should generate monthly report for January', () => {
        vi.useFakeTimers();
        vi.setSystemTime(new Date('2025-01-15T10:00:00Z'));

        const report = service.generateMonthlyReport();

        expect(report.startDate).toBe('2025-01-01');
        expect(report.endDate).toBe('2025-01-31');
    });

    it('should handle February in leap year', () => {
        vi.useFakeTimers();
        vi.setSystemTime(new Date('2024-02-15T10:00:00Z')); // 2024 is leap year

        const report = service.generateMonthlyReport();

        expect(report.endDate).toBe('2024-02-29');
    });

    it('should handle February in non-leap year', () => {
        vi.useFakeTimers();
        vi.setSystemTime(new Date('2025-02-15T10:00:00Z')); // 2025 is not leap year

        const report = service.generateMonthlyReport();

        expect(report.endDate).toBe('2025-02-28');
    });
});
```

---

## Guidelines

1. **Always use `vi.useFakeTimers()`** - Never rely on real time in tests
2. **Always clean up in `afterEach`** - Call `vi.useRealTimers()` even for single tests
3. **Use hard-coded timestamps** - Don't compute expected values
4. **Use `vi.advanceTimersByTime()`** - For testing time-dependent behavior
5. **Test edge cases** - Month boundaries, leap years, timezone transitions
6. **Use ISO date strings** - `new Date('2025-01-15T10:00:00Z')` for clarity

### Benefits

✅ **Deterministic tests** - Same result every run
✅ **No flaky tests** - Time doesn't drift
✅ **Testable edge cases** - Specific dates easily tested
✅ **Fast tests** - No waiting for real time to pass
✅ **Clear intent** - Fixed time makes test purpose obvious

---
