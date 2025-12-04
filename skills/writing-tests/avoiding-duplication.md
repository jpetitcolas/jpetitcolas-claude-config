# Testing Patterns - Avoiding Duplication

Patterns for avoiding duplicated tests and consolidating related assertions.

## Table of Contents

1. [Why Avoid Duplication](#why-avoid-duplication)
2. [Consolidate Tests for Same Behavior](#consolidate-tests-for-same-behavior)
3. [When Tests Are NOT Duplicated](#when-tests-are-not-duplicated)
4. [Guidelines](#guidelines)

---

## Why Avoid Duplication

**Principle:** Each test should verify a unique behavior. No overlap.

**Benefits:**
- **Faster test suite** - No redundant executions
- **Easier maintenance** - Change in one place
- **Clearer coverage** - Each test has distinct purpose
- **Better signal** - Failures point to specific issues

---

## Consolidate Tests for Same Behavior

### Don't: Separate Tests for Different Properties of Same Behavior

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
```

### Do: Consolidate into Single Test

```typescript
// ✅ DO: Consolidate into single test
it('should save user to database', () => {
    const user = { email: 'test@example.com', name: 'Alice' };
    service.save(user);
    expect(mockDb.save).toHaveBeenCalledWith(user);
});
```

---

## When Tests Are NOT Duplicated

Tests are **not** duplicated when they verify:

### 1. Different Behaviors (Same Input)

```typescript
// ✅ These are NOT duplicated - different behaviors
it('should save user to database', async () => {
    await service.createUser(userData);
    expect(mockDb.save).toHaveBeenCalledWith(userData);
});

it('should send welcome email', async () => {
    await service.createUser(userData);
    expect(mockEmailer.sendWelcome).toHaveBeenCalledWith(userData.email);
});
```

### 2. Different Conditions (Same Method)

```typescript
// ✅ These are NOT duplicated - different conditions
describe('when user exists', () => {
    it('should throw conflict error', async () => {
        mockDb.findByEmail.mockResolvedValue(existingUser);
        await expect(service.createUser(userData)).rejects.toThrow('User exists');
    });
});

describe('when user does not exist', () => {
    it('should create user', async () => {
        mockDb.findByEmail.mockResolvedValue(null);
        await service.createUser(userData);
        expect(mockDb.save).toHaveBeenCalled();
    });
});
```

### 3. Data Variations (Use `it.each` Instead)

```typescript
// ✅ Data variations should use it.each, not separate tests
// See: it-each.md for patterns
it.each([
    { input: 'SAVE10', expected: 90 },
    { input: 'SAVE20', expected: 80 },
    { input: null, expected: 100 },
])('should calculate discount: coupon=$input', ({ input, expected }) => {
    expect(calculateDiscount(100, input)).toBe(expected);
});
```

---

## Guidelines

1. **One test per unique behavior** - Don't split properties into separate tests
2. **Consolidate related assertions** - Test full object, not individual fields
3. **Use `it.each` for data variations** - Same logic, different inputs
4. **Different conditions = different tests** - Not duplication
5. **Different side effects = different tests** - Not duplication

### Code Smells

- Multiple tests with identical setup
- Tests that only differ in which property they check
- Copy-pasted test code with minor changes

### Benefits

✅ **Faster tests** - No redundant executions
✅ **Easier maintenance** - Single source of truth
✅ **Clearer failures** - Each test has unique purpose
✅ **Better coverage** - No false sense of coverage from duplicates

---
