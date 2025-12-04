# Testing Patterns - Focused Assertions

Patterns for testing only what the test description says and avoiding over-testing.

## Table of Contents

1. [Why Focus Assertions](#why-focus-assertions)
2. [Test Only What the Description Says](#test-only-what-the-description-says)
3. [Use Matchers for Irrelevant Arguments](#use-matchers-for-irrelevant-arguments)
4. [Split Independent Behaviors](#split-independent-behaviors)
5. [Guidelines](#guidelines)

---

## Why Focus Assertions

**Principle:** Test ONLY what the description says. Minimize unnecessary assertions.

**Benefits:**
- **Clear intent** - Test name matches test content
- **Focused failures** - Know exactly what broke
- **Less brittleness** - Changes to unrelated code don't break tests
- **Self-documenting** - Test describes one specific behavior

---

## Test Only What the Description Says

### Don't: Add Assertions Beyond the Description

```typescript
// ❌ DON'T: Add assertions beyond what the test name describes
it('should return 404 status code', () => {
    const response = controller.getUser('invalid-id');
    expect(response.statusCode).toBe(404); // ✓ Matches description
    expect(response.body).toEqual({ error: 'Not found' }); // ✗ Extra
    expect(mockLogger.warn).toHaveBeenCalled(); // ✗ Not in description
});
```

### Do: Test Only What's Described

```typescript
// ✅ DO: Test only what the description says
it('should return 404 status code', () => {
    const response = controller.getUser('invalid-id');
    expect(response.statusCode).toBe(404);
});

// If you need to test the body and logging, create separate tests:
it('should return error message in body', () => {
    const response = controller.getUser('invalid-id');
    expect(response.body).toEqual({ error: 'Not found' });
});

it('should log warning for invalid user request', () => {
    controller.getUser('invalid-id');
    expect(mockLogger.warn).toHaveBeenCalled();
});
```

---

## Use Matchers for Irrelevant Arguments

### Don't: Assert All Arguments When Only One Matters

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
```

### Do: Use Matchers for Arguments Not in Description

```typescript
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

---

## Split Independent Behaviors

### Don't: Test Multiple Independent Behaviors in One Test

```typescript
// ❌ DON'T: Test multiple independent behaviors
it('should process order correctly', async () => {
    await service.processOrder(order);

    // These are INDEPENDENT behaviors that should be separate tests
    expect(mockPayment.charge).toHaveBeenCalledWith(order.total);
    expect(mockInventory.reserve).toHaveBeenCalledWith(order.items);
    expect(mockEmail.sendConfirmation).toHaveBeenCalledWith(order.email);
    expect(mockAnalytics.track).toHaveBeenCalledWith('order_processed');
});
```

### Do: Split into Separate Tests

```typescript
// ✅ DO: Split independent behaviors into separate tests
describe('processOrder', () => {
    it('should charge payment', async () => {
        await service.processOrder(order);
        expect(mockPayment.charge).toHaveBeenCalledWith(order.total);
    });

    it('should reserve inventory', async () => {
        await service.processOrder(order);
        expect(mockInventory.reserve).toHaveBeenCalledWith(order.items);
    });

    it('should send confirmation email', async () => {
        await service.processOrder(order);
        expect(mockEmail.sendConfirmation).toHaveBeenCalledWith(order.email);
    });

    it('should track analytics event', async () => {
        await service.processOrder(order);
        expect(mockAnalytics.track).toHaveBeenCalledWith('order_processed');
    });
});
```

---

## Guidelines

1. **Match description** - Assertions should match what the test name says
2. **One behavior per test** - Split independent behaviors
3. **Use matchers** - `expect.any()` for irrelevant arguments
4. **Rename if needed** - If you need more assertions, rename the test
5. **Ask yourself** - "If this assertion fails, does the test name describe why?"

### Signs of Over-Testing

- Test has assertions unrelated to its name
- Test checks implementation details not in description
- Test would fail for changes unrelated to its purpose
- Test name is vague to justify many assertions

### Benefits

✅ **Clear failures** - Know exactly what broke
✅ **Focused tests** - Each test has one purpose
✅ **Less brittleness** - Unrelated changes don't break tests
✅ **Self-documenting** - Test name describes the behavior

---
