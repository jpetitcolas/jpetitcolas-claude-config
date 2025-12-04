# Testing Patterns - Behavior Testing

Patterns for testing behaviors instead of implementation details.

## Table of Contents

1. [Why Test Behaviors](#why-test-behaviors)
2. [Test Observable Outcomes](#test-observable-outcomes)
3. [Don't Test Internal Mechanics](#dont-test-internal-mechanics)
4. [Test Side Effects Appropriately](#test-side-effects-appropriately)
5. [The Refactoring Test](#the-refactoring-test)
6. [Guidelines](#guidelines)

---

## Why Test Behaviors

**Principle:** Test what the code does (behavior), not how it does it (implementation).

**Benefits:**
- **Survives refactoring** - Change implementation, tests still pass
- **Documents behavior** - Tests show what the system does
- **Less brittle** - Internal changes don't break tests
- **Better coverage** - Focus on user-facing outcomes

---

## Test Observable Outcomes

### Don't: Test Internal Mechanics and Call Order

```typescript
// ❌ DON'T: Test internal mechanics and call order
it('should call validateEmail then saveToDb', async () => {
    const spy1 = vi.spyOn(service, 'validateEmail');
    const spy2 = vi.spyOn(service, 'saveToDb');

    await service.registerUser({ email: 'user@example.com' });

    expect(spy1).toHaveBeenCalledBefore(spy2);
    // Breaks if you change internal order
});
```

### Do: Test Observable Outcomes

```typescript
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

---

## Don't Test Internal Mechanics

### Don't: Spy on Internal Methods

```typescript
// ❌ DON'T: Spy on your own internal methods
it('should validate before saving', async () => {
    const validateSpy = vi.spyOn(service, 'validateData');
    const transformSpy = vi.spyOn(service, 'transformData');

    await service.process(data);

    expect(validateSpy).toHaveBeenCalled();
    expect(transformSpy).toHaveBeenCalled();
});
```

### Do: Test the Final Result

```typescript
// ✅ DO: Test what the user sees or what side effects occur
it('should process valid data and store result', async () => {
    const result = await service.process(validData);

    expect(result.status).toBe('success');
    expect(mockDb.save).toHaveBeenCalledWith(
        expect.objectContaining({ processed: true })
    );
});

it('should reject invalid data with error message', async () => {
    await expect(service.process(invalidData)).rejects.toThrow(
        'Validation failed: email is required'
    );
});
```

---

## Test Side Effects Appropriately

### When to Test Side Effects

Side effects ARE part of behavior when they're externally observable:

```typescript
// ✅ Good: Testing external side effects
it('should send notification when order is placed', async () => {
    await service.placeOrder(order);

    // External service call IS the behavior
    expect(mockNotificationService.send).toHaveBeenCalledWith({
        userId: order.userId,
        message: expect.stringContaining('Order confirmed')
    });
});

it('should persist order to database', async () => {
    await service.placeOrder(order);

    // Database write IS the behavior
    expect(mockDb.orders.create).toHaveBeenCalledWith(
        expect.objectContaining({
            id: expect.any(String),
            items: order.items,
            total: order.total
        })
    );
});
```

### What NOT to Test

```typescript
// ❌ DON'T: Test internal state or private method calls
it('should set internal flag', async () => {
    await service.process(data);
    expect(service._internalState.processed).toBe(true); // Private state!
});

// ❌ DON'T: Test helper function calls within the class
it('should call formatPrice internally', async () => {
    const spy = vi.spyOn(service, 'formatPrice');
    await service.calculateTotal(items);
    expect(spy).toHaveBeenCalled(); // Implementation detail!
});
```

---

## The Refactoring Test

**Ask yourself:** "If I completely refactor the implementation, should this test still pass?"

```typescript
// Original implementation
class OrderService {
    async calculateTotal(items: Item[]): Promise<number> {
        const subtotal = this.sumItems(items);
        const tax = this.calculateTax(subtotal);
        return subtotal + tax;
    }
}

// Refactored implementation (same behavior, different internals)
class OrderService {
    async calculateTotal(items: Item[]): Promise<number> {
        return items.reduce((sum, item) => {
            const itemTotal = item.price * item.quantity;
            const itemTax = itemTotal * 0.1;
            return sum + itemTotal + itemTax;
        }, 0);
    }
}

// ✅ Good test - survives refactoring
it('should calculate total including 10% tax', async () => {
    const items = [
        { price: 10, quantity: 2 },
        { price: 5, quantity: 1 }
    ];

    const result = await service.calculateTotal(items);

    expect(result).toBe(27.5); // 25 + 2.5 tax
});

// ❌ Bad test - breaks on refactoring
it('should call sumItems and calculateTax', async () => {
    const sumSpy = vi.spyOn(service, 'sumItems');
    const taxSpy = vi.spyOn(service, 'calculateTax');

    await service.calculateTotal(items);

    expect(sumSpy).toHaveBeenCalled(); // Fails after refactoring!
    expect(taxSpy).toHaveBeenCalled(); // Fails after refactoring!
});
```

---

## Guidelines

1. **Test public interfaces** - Not private methods
2. **Assert on results** - Not on how you got there
3. **Verify side effects** - When they're externally observable
4. **Mock external dependencies** - Not your own code
5. **Name tests by behavior** - What it does, not how
6. **Refactoring test** - Would this test survive a rewrite?

### What to Test

- ✅ Return values
- ✅ Thrown exceptions
- ✅ External service calls (APIs, databases)
- ✅ Published events
- ✅ State changes visible to users

### What NOT to Test

- ❌ Private method calls
- ❌ Internal state
- ❌ Call order of internal methods
- ❌ Implementation-specific helpers
- ❌ How code is structured internally

### Benefits

✅ **Survives refactoring** - Tests don't break on implementation changes
✅ **Documents behavior** - Tests show what system does for users
✅ **Less maintenance** - Fewer tests to update on changes
✅ **Better design** - Forces thinking about observable outcomes

---
