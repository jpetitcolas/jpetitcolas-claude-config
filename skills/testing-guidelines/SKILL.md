---
name: testing-guidelines
description: Testing best practices for JavaScript/TypeScript with Vitest, including time mocking, focused assertions, avoiding duplicated tests, and focusing on behaviors over implementation details
---

# Testing Guidelines

## Purpose

Provides comprehensive testing best practices for JavaScript/TypeScript projects using Vitest, ensuring tests are maintainable, focused, non-duplicated, and follow consistent patterns.

## When to Use

Activate this skill when:
- Writing new test files or test cases
- Refactoring existing tests
- Reviewing test code
- Setting up test mocks or fixtures
- Debugging failing tests
- Improving test clarity and maintainability

---

## Core Principles

### 1. Never Use `new Date()` - Mock Time Instead

**Problem:** Tests using `new Date()` are non-deterministic and fail randomly.

**Scenario: Testing timestamp values**

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

### 2. Hard-Write Values in Assertions

**Principle:** Expected values should be explicit, not computed.

**Rule:** Hard-code expected values. For large objects (>3 properties), use the variable with spread syntax to avoid duplication.

**Scenario 1: Don't compute values**

```typescript
// ❌ DON'T: Compute expected values
it('should calculate total price', () => {
    const result = calculateOrder(items);
    expect(result.total).toBe(items.length * price); // Computed!
});

// ✅ DO: Hard-code the expected value
it('should calculate total price', () => {
    const result = calculateOrder(items);
    expect(result.total).toBe(299.97); // Explicit value
});
```

**Scenario 2: Don't use unnecessary variables**

```typescript
// ❌ DON'T: Use variables when values are known
it('should create user with correct id', async () => {
    const expectedId = 'user-123';
    const user = await service.createUser(data);
    expect(user.id).toBe(expectedId); // Unnecessary variable
});

// ✅ DO: Hard-code directly
it('should create user with correct id', async () => {
    const user = await service.createUser(data);
    expect(user.id).toBe('user-123'); // Direct value
});
```

**Scenario 3: Small vs large objects**

```typescript
// ❌ DON'T: Test only one property when you should verify the full object
const userData = { email: 'test@example.com', name: 'Alice' };
const user = await service.createUser(userData);
expect(user.email).toBe(userData.email); // Only checks one field

// ✅ DO: Hard-code small objects (≤3 properties)
expect(user).toEqual({
    id: 'user-123',
    email: 'test@example.com',
    name: 'Alice'
});

// ✅ DO: For large objects (>3 properties), use the variable + matchers
const userData = { email: 'test@example.com', name: 'Alice', phone: '555-1234' };
const user = await service.createUser(userData);
expect(user).toEqual({
    ...userData, // Spread the variable (avoids duplicating many fields)
    id: expect.any(String), // Use matchers for generated fields
    createdAt: expect.any(Number),
    status: 'active' // Hard-code known defaults
});
```

**Also: Prefer one precise assertion over multiple weak ones:**

```typescript
// ❌ DON'T: Multiple weak assertions
expect(result).toHaveLength(2);
expect(result).toContainEqual(item1);
expect(result).toContainEqual(item2);

// ✅ DO: Single precise assertion
expect(result).toEqual([item1, item2]);
```

---

### 3. Avoid Duplicated Tests

**Principle:** Each test should verify a unique behavior. No overlap.

**Scenario: Multiple tests checking different parts of same behavior**

```typescript
// ❌ DON'T: Write separate tests for different properties of same behavior
it('should save user with email', () => {
    const user = { email: 'test@example.com', name: 'Alice' };
    service.save(user);
    expect(mockDb.save).toHaveBeenCalledWith(
        expect.objectContaining({ email: 'test@example.com' })
    );
});

it('should save user with name', () => {
    const user = { email: 'test@example.com', name: 'Alice' };
    service.save(user);
    expect(mockDb.save).toHaveBeenCalledWith(
        expect.objectContaining({ name: 'Alice' })
    );
});

// ✅ DO: Consolidate into single test
it('should save user to database', () => {
    const user = { email: 'test@example.com', name: 'Alice' };
    service.save(user);
    expect(mockDb.save).toHaveBeenCalledWith(user);
});
```

**Note:** For data variations (same logic, different inputs), see Principle #5 - Use `it.each`.

---

### 4. Focus Tests on Assertion Description

**Principle:** Test ONLY what the description says. Minimize unnecessary assertions.

**Scenario 1: Testing more than the description says**

```typescript
// ❌ DON'T: Add assertions beyond what the test name describes
it('should return 404 status code', () => {
    const response = controller.getUser('invalid-id');
    expect(response.statusCode).toBe(404); // ✓ Matches description
    expect(response.body).toEqual({ error: 'Not found' }); // ✗ Extra
    expect(mockLogger.warn).toHaveBeenCalled(); // ✗ Not in description
});

// ✅ DO: Test only what the description says
it('should return 404 status code', () => {
    const response = controller.getUser('invalid-id');
    expect(response.statusCode).toBe(404);
});
```

**Scenario 2: Testing all arguments when only one matters**

```typescript
// ❌ DON'T: Assert all arguments when description only mentions one
it('should use deployment_id as the key identifier', async () => {
    await service.recordStart({ deployment_id: 'custom-id-789' });

    expect(mockRedis.setex).toHaveBeenCalledWith(
        'test:custom-id-789:data',
        3600, // ✗ Testing TTL (not in description)
        '{"deployment_id":"custom-id-789"}' // ✗ Testing value (not in description)
    );
});

// ✅ DO: Use matchers for arguments not mentioned in description
it('should use deployment_id as the key identifier', async () => {
    const metadata = { deployment_id: 'custom-id-789', environment_id: 'env-456' };
    await service.recordStart(metadata);

    expect(mockRedis.setex).toHaveBeenCalledWith(
        'test:custom-id-789:data', // Only test the key
        expect.any(Number),
        expect.any(String)
    );
});
```

**Important:** If a test verifies multiple INDEPENDENT behaviors, split them into separate tests. Each test should have one clear intent.

---

### 5. Use `it.each` for Data Variations

**When to Use:** Tests where only input data changes, but logic is identical.

**Scenario: Testing with multiple input/output pairs**

```typescript
// ❌ DON'T: Write separate test for each data variation
it('should calculate discount with SAVE10 coupon', () => {
    expect(calculateDiscount(100, 'SAVE10')).toBe(90);
});

it('should calculate discount with SAVE20 coupon', () => {
    expect(calculateDiscount(100, 'SAVE20')).toBe(80);
});

it('should calculate discount with no coupon', () => {
    expect(calculateDiscount(50, null)).toBe(50);
});

// ✅ DO: Use it.each to test all variations in one place
it.each([
    { price: 100, coupon: 'SAVE10', expected: 90 },
    { price: 100, coupon: 'SAVE20', expected: 80 },
    { price: 50, coupon: 'SAVE10', expected: 45 },
    { price: 50, coupon: null, expected: 50 },
])('should calculate: price=$price, coupon=$coupon', ({ price, coupon, expected }) => {
    expect(calculateDiscount(price, coupon)).toBe(expected);
});
```

---

### 6. Focus on Behaviors, Not Implementation Details

**Principle:** Test what the code does (behavior), not how it does it (implementation).

**Scenario: Testing user registration**

```typescript
// ❌ DON'T: Test internal mechanics and call order
it('should call validateEmail then saveToDb', async () => {
    const spy1 = vi.spyOn(service, 'validateEmail');
    const spy2 = vi.spyOn(service, 'saveToDb');

    await service.registerUser({ email: 'user@example.com' });

    expect(spy1).toHaveBeenCalledBefore(spy2);
    // Breaks if you change internal order
});

// ✅ DO: Test observable outcomes and side effects
it('should register user and send welcome email', async () => {
    await service.registerUser({ email: 'user@example.com' });

    const user = await db.findUserByEmail('user@example.com');
    expect(user).toBeDefined();
    expect(mockEmailService.send).toHaveBeenCalledWith(
        expect.objectContaining({
            to: 'user@example.com',
            subject: expect.stringContaining('Welcome')
        })
    );
    // Survives internal refactoring
});
```

**Guidelines:**
1. Test public interfaces, not private methods
2. Assert on results, not on how you got there
3. Verify side effects (API calls, database writes) when appropriate
4. Mock external dependencies, not your own code
5. Name tests by behavior (what), not implementation (how)
6. Ask: "If I refactor completely, should this test still pass?"

---

## Quick Reference Checklist

When writing tests, ensure:

- [ ] Time is mocked with `vi.useFakeTimers()` - no `new Date()`
- [ ] Environment variables mocked with `vi.stubEnv()` - no direct `process.env` mutation
- [ ] Expected values are hard-coded (except large objects >3 props)
- [ ] Using precise assertions (`toEqual`) instead of multiple weak ones
- [ ] No duplicate tests - each test verifies unique behavior
- [ ] Shared test data extracted to avoid duplication (DRY)
- [ ] Assertions match test description - only test what's described
- [ ] Split tests when verifying multiple independent behaviors
- [ ] Used `it.each` when only data varies
- [ ] Tests focus on behavior (what), not implementation (how)
- [ ] Testing public interfaces, not private methods or internal state
- [ ] Exception messages are tested (not just exception type)
- [ ] Mock calls verified with `toHaveBeenCalledWith()` (not just `toHaveBeenCalled()`)
- [ ] Test names are descriptive and specific
- [ ] Test methods ordered same as source file for easy navigation
- [ ] Timers cleaned up with `vi.useRealTimers()` in `afterEach`
- [ ] Environment variables cleaned up with `vi.unstubAllEnvs()` in `afterEach`
- [ ] Tests are independent and can run in any order

---

## Reference Files

### Additional Patterns (Beyond the 6 core principles)
- [flow.md](resources/patterns/flow.md) - Test ordering (method order, code flow)
- [organization.md](resources/patterns/organization.md) - Nested describe blocks, grouping by condition
- [method-visibility.md](resources/patterns/method-visibility.md) - Private, protected, and static methods
- [dry.md](resources/patterns/dry.md) - Extract shared data, object spread, fixtures
- [mocking.md](resources/patterns/mocking.md) - Mocks, async functions, error cases, environment variables
- [logs.md](resources/patterns/logs.md) - Log levels, structured logs, sensitive data

---

## Common Pitfalls

**Avoid these common mistakes:**

1. **Using `new Date()`** - Always mock time
2. **Mutating `process.env` directly** - Use `vi.stubEnv()` and `vi.unstubAllEnvs()`
3. **Inline cleanup** - Use `afterEach` for timers/env vars (runs even if test throws)
4. **Computing expected values** - Hard-code them
5. **Multiple weak assertions** - Use one precise assertion
6. **Duplicate tests** - Consolidate or use `it.each`
7. **Duplicating test data** - Extract shared constants to avoid repetition
8. **Duplicating setup across tests** - Use nested `describe` to group by condition
9. **Testing too much** - Focus on test description
10. **Testing multiple independent behaviors in one test** - Split them
11. **Testing implementation** - Focus on behavior
12. **Accessing private state** - Test public API only
13. **Testing exception type only** - Always test exception messages too
14. **Using `toHaveBeenCalled()` only** - Use `toHaveBeenCalledWith()` for precision
15. **Testing abstract classes via production subclasses** - Create test-specific implementations
16. **Random test order** - Match source file method order for maintainability
17. **Dependent tests** - Each test should be independent
18. **Vague test names** - Be specific and descriptive

---

## Quick Start Example

Here's a complete test file following all principles:

```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';
import { UserService } from './user-service';

describe('UserService', () => {
    let service: UserService;
    let mockDb: any;
    let mockEmailer: any;

    beforeEach(() => {
        // Mock time
        vi.useFakeTimers();
        vi.setSystemTime(new Date('2025-05-05 12:00:00'));

        // Setup mocks
        mockDb = { save: vi.fn(), find: vi.fn() };
        mockEmailer = { send: vi.fn() };

        service = new UserService(mockDb, mockEmailer);
    });

    afterEach(() => {
        vi.useRealTimers();
    });

    // Behavior-focused test
    it('should create user with welcome email', async () => {
        const userData = { email: 'alice@example.com', name: 'Alice' };

        await service.createUser(userData);

        // Hard-coded expected values
        expect(mockDb.save).toHaveBeenCalledWith({
            email: 'alice@example.com',
            name: 'Alice',
            createdAt: 1746446400000 // Mocked time
        });

        expect(mockEmailer.send).toHaveBeenCalledWith(
            expect.objectContaining({
                to: 'alice@example.com',
                subject: expect.stringContaining('Welcome')
            })
        );
    });

    // Using it.each for data variations
    describe('validation', () => {
        it.each([
            { email: 'valid@example.com', expected: true },
            { email: 'invalid', expected: false },
            { email: '', expected: false },
            { email: '@example.com', expected: false },
        ])('should validate email: $email', ({ email, expected }) => {
            expect(service.isValidEmail(email)).toBe(expected);
        });
    });
});
```

---

## Related Documentation

- [Vitest Documentation](https://vitest.dev/)
- [Vitest Mocking Guide](https://vitest.dev/guide/mocking.html)
- [Vitest API Reference](https://vitest.dev/api/)

---

**Last Updated:** 2025-01-10
**Lines:** < 500 (following Anthropic best practices)
