# Testing Patterns - Using it.each

Patterns for testing data variations with `it.each` instead of duplicating tests.

## Table of Contents

1. [When to Use it.each](#when-to-use-iteach)
2. [Basic Pattern](#basic-pattern)
3. [Object Syntax](#object-syntax)
4. [Multiple Parameters](#multiple-parameters)
5. [Edge Cases and Boundaries](#edge-cases-and-boundaries)
6. [When NOT to Use it.each](#when-not-to-use-iteach)
7. [Guidelines](#guidelines)

---

## When to Use it.each

**Use `it.each` when:** Tests where only input data changes, but logic is identical.

**Benefits:**
- **DRY** - No duplicated test code
- **Easy to add cases** - Just add a row to the table
- **Clear data coverage** - All variations visible at once
- **Maintainable** - Change test logic in one place

---

## Basic Pattern

### Don't: Separate Tests for Each Data Variation

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
```

### Do: Use it.each

```typescript
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

## Object Syntax

Use object syntax for clarity when you have multiple parameters:

```typescript
it.each([
    { input: 'hello', expected: 'HELLO' },
    { input: 'World', expected: 'WORLD' },
    { input: '', expected: '' },
    { input: '123', expected: '123' },
])('should uppercase "$input" to "$expected"', ({ input, expected }) => {
    expect(toUpperCase(input)).toBe(expected);
});
```

---

## Multiple Parameters

### Testing Validation Rules

```typescript
it.each([
    { email: '', error: 'Email is required' },
    { email: 'invalid', error: 'Invalid email format' },
    { email: 'no@domain', error: 'Invalid email format' },
    { email: 'test@.com', error: 'Invalid email format' },
])('should reject invalid email: "$email"', ({ email, error }) => {
    expect(() => validateEmail(email)).toThrow(error);
});
```

### Testing Status Codes

```typescript
it.each([
    { status: 200, isSuccess: true },
    { status: 201, isSuccess: true },
    { status: 204, isSuccess: true },
    { status: 400, isSuccess: false },
    { status: 404, isSuccess: false },
    { status: 500, isSuccess: false },
])('should return isSuccess=$isSuccess for status $status', ({ status, isSuccess }) => {
    expect(isSuccessStatus(status)).toBe(isSuccess);
});
```

---

## Edge Cases and Boundaries

Use `it.each` to systematically test boundaries:

```typescript
describe('isAdult', () => {
    it.each([
        { age: 17, expected: false },
        { age: 18, expected: true },
        { age: 19, expected: true },
        { age: 0, expected: false },
        { age: -1, expected: false },
        { age: 100, expected: true },
    ])('should return $expected for age $age', ({ age, expected }) => {
        expect(isAdult(age)).toBe(expected);
    });
});
```

---

## When NOT to Use it.each

### Don't: When Tests Have Different Logic

```typescript
// ❌ DON'T: Force it.each when test logic differs
it.each([
    {
        scenario: 'valid user',
        setup: () => mockDb.findUser.mockResolvedValue(user),
        action: () => service.getUser('123'),
        assertion: (result) => expect(result).toEqual(user)
    },
    // This is over-engineered!
])('should handle $scenario', async ({ setup, action, assertion }) => {
    setup();
    const result = await action();
    assertion(result);
});

// ✅ DO: Use regular tests when logic differs
it('should return user when found', async () => {
    mockDb.findUser.mockResolvedValue(user);
    const result = await service.getUser('123');
    expect(result).toEqual(user);
});

it('should throw when user not found', async () => {
    mockDb.findUser.mockResolvedValue(null);
    await expect(service.getUser('123')).rejects.toThrow('Not found');
});
```

### Don't: When Only One or Two Cases

```typescript
// ❌ DON'T: Overkill for just 1-2 cases
it.each([
    { input: true, expected: false },
])('should negate $input', ({ input, expected }) => {
    expect(negate(input)).toBe(expected);
});

// ✅ DO: Just write a simple test
it('should negate boolean', () => {
    expect(negate(true)).toBe(false);
    expect(negate(false)).toBe(true);
});
```

---

## Guidelines

1. **Use for data variations** - Same logic, different inputs
2. **Use object syntax** - Clearer than array syntax
3. **Include descriptive test names** - Use `$property` interpolation
4. **Group related variations** - All cases for one behavior
5. **Don't force it** - Regular tests are fine when logic differs
6. **Minimum 3 cases** - Less than 3? Just write regular tests

### Signs You Should Use it.each

- Copy-pasting tests with only data changes
- Testing boundary conditions (min, max, edge cases)
- Testing multiple valid inputs
- Testing multiple error conditions

### Benefits

✅ **DRY** - No duplicated test code
✅ **Easy to extend** - Add new cases as rows
✅ **Clear coverage** - All variations visible together
✅ **Maintainable** - Single source of test logic

---
