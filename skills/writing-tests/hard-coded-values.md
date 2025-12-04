# Testing Patterns - Hard-Coded Values

Patterns for writing explicit, hard-coded expected values in assertions.

## Table of Contents

1. [Why Hard-Code Values](#why-hard-code-values)
2. [Don't Compute Expected Values](#dont-compute-expected-values)
3. [Don't Use Unnecessary Variables](#dont-use-unnecessary-variables)
4. [Small vs Large Objects](#small-vs-large-objects)
5. [Prefer Precise Assertions](#prefer-precise-assertions)
6. [Guidelines](#guidelines)

---

## Why Hard-Code Values

**Principle:** Expected values should be explicit, not computed.

**Benefits:**
- **Clear intent** - See exactly what's expected
- **Catches bugs** - Computed values can have the same bug as the code
- **Self-documenting** - Test shows the expected outcome
- **Easier debugging** - Know exactly what failed

---

## Don't Compute Expected Values

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

---

## Don't Use Unnecessary Variables

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

---

## Small vs Large Objects

**Rule:** Hard-code expected values. For large objects (>3 properties), use the variable with spread syntax to avoid duplication.

### Don't: Test Only One Property

```typescript
// ❌ DON'T: Test only one property when you should verify the full object
const userData = { email: 'test@example.com', name: 'Alice' };
const user = await service.createUser(userData);
expect(user.email).toBe(userData.email); // Only checks one field
```

### Do: Hard-Code Small Objects (≤3 properties)

```typescript
// ✅ DO: Hard-code small objects
expect(user).toEqual({
    id: 'user-123',
    email: 'test@example.com',
    name: 'Alice'
});
```

### Do: Use Spread for Large Objects (>3 properties)

```typescript
// ✅ DO: For large objects, use the variable + matchers
const userData = { email: 'test@example.com', name: 'Alice', phone: '555-1234' };
const user = await service.createUser(userData);
expect(user).toEqual({
    ...userData, // Spread the variable (avoids duplicating many fields)
    id: expect.any(String), // Use matchers for generated fields
    createdAt: expect.any(Number),
    status: 'active' // Hard-code known defaults
});
```

---

## Prefer Precise Assertions

**Principle:** One precise assertion is better than multiple weak ones.

```typescript
// ❌ DON'T: Multiple weak assertions
expect(result).toHaveLength(2);
expect(result).toContainEqual(item1);
expect(result).toContainEqual(item2);

// ✅ DO: Single precise assertion
expect(result).toEqual([item1, item2]);
```

---

## Guidelines

1. **Hard-code expected values** - Don't compute them
2. **Avoid unnecessary variables** - Inline simple values
3. **Small objects (≤3 props)** - Hard-code entirely
4. **Large objects (>3 props)** - Use spread + matchers
5. **One precise assertion** - Better than multiple weak ones
6. **Use `expect.any()`** - For generated values (IDs, timestamps)

### Benefits

✅ **Clear expectations** - See exactly what should happen
✅ **Catches computation bugs** - Won't replicate code errors
✅ **Self-documenting** - Test shows expected outcomes
✅ **Easier debugging** - Know exactly what value failed

---
