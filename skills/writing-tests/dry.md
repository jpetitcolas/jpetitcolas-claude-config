# Testing Patterns - DRY (Don't Repeat Yourself)

Patterns for extracting and reusing shared test data to avoid duplication.

## Table of Contents

1. [Don't Repeat Test Data](#dont-repeat-test-data-across-multiple-tests)
2. [Shared Constants at Describe Level](#pattern-shared-constants-at-describe-level)
3. [Object Spread for Variations](#pattern-object-spread-for-variations)
4. [Shared Fixtures in beforeEach](#pattern-shared-fixtures-in-beforeeach)
5. [When to Extract vs Keep Inline](#when-to-extract-vs-keep-inline)

---

## Extract Shared Test Data (DRY)

### Don't Repeat Test Data Across Multiple Tests

**Principle:** When the same data/constants appear in multiple tests, extract them to avoid repetition.

**Why:**
- **Single source of truth** - Change in one place
- **Easier maintenance** - Update data structure once
- **Reduces errors** - No copy-paste mistakes
- **Clearer intent** - Shared data is explicitly shared

### Example: Extract Repeated Data

```typescript
// ❌ Bad: Repeated data across tests
describe('StaleEventController', () => {
    it('should send incomplete metrics for stale events', async () => {
        const staleEvent = {
            environment_id: 'env-123',
            environment_name: 'preview-pr-42',
            application_name: 'hermes-api',
            deployment_id: 'deploy-stale',
            timestamp: FIXED_TIME - 2000,
        };

        vi.mocked(mockTrackingService.findStale).mockResolvedValue([staleEvent]);

        await controller.detectAndReportStaleEvents();

        expect(mockDatadogService.sendLifecycleIncompleteMetric).toHaveBeenCalledWith(
            'deployment',
            {
                environment_id: 'env-123',
                environment_name: 'preview-pr-42',
                application_name: 'hermes-api',
                deployment_id: 'deploy-stale',
                timestamp: FIXED_TIME,
                duration_seconds: 2000,
            }
        );
    });

    it('should clear stale event data', async () => {
        // ❌ Same data duplicated!
        const staleEvent = {
            environment_id: 'env-123',
            environment_name: 'preview-pr-42',
            application_name: 'hermes-api',
            deployment_id: 'deploy-stale',
            timestamp: FIXED_TIME - 2000,
        };

        vi.mocked(mockTrackingService.findStale).mockResolvedValue([staleEvent]);

        await controller.detectAndReportStaleEvents();

        expect(mockTrackingService.clearData).toHaveBeenCalledWith('deploy-stale');
    });
});

// ✅ Good: Extract shared data
describe('StaleEventController', () => {
    // Shared test data at describe level
    const staleEvent = {
        environment_id: 'env-123',
        environment_name: 'preview-pr-42',
        application_name: 'hermes-api',
        deployment_id: 'deploy-stale',
        timestamp: FIXED_TIME - 2000,
    };

    beforeEach(() => {
        vi.mocked(mockTrackingService.findStale).mockResolvedValue([staleEvent]);
    });

    it('should send incomplete metrics for stale events', async () => {
        await controller.detectAndReportStaleEvents();

        expect(mockDatadogService.sendLifecycleIncompleteMetric).toHaveBeenCalledWith(
            'deployment',
            {
                environment_id: 'env-123',
                environment_name: 'preview-pr-42',
                application_name: 'hermes-api',
                deployment_id: 'deploy-stale',
                timestamp: FIXED_TIME,
                duration_seconds: 2000,
            }
        );
    });

    it('should clear stale event data', async () => {
        await controller.detectAndReportStaleEvents();

        expect(mockTrackingService.clearData).toHaveBeenCalledWith('deploy-stale');
    });
});
```

### Pattern: Shared Constants at Describe Level

```typescript
describe('UserService', () => {
    // ✅ Extract shared test data
    const validUser = {
        email: 'alice@example.com',
        name: 'Alice',
        age: 30,
    };

    const invalidUser = {
        email: 'invalid-email',
        name: '',
    };

    describe('createUser', () => {
        it('should create user with valid data', async () => {
            await service.createUser(validUser);
            expect(mockDb.save).toHaveBeenCalledWith(validUser);
        });

        it('should reject invalid user data', async () => {
            await expect(service.createUser(invalidUser)).rejects.toThrow();
        });
    });

    describe('updateUser', () => {
        it('should update user with valid data', async () => {
            await service.updateUser('user-123', validUser);
            expect(mockDb.update).toHaveBeenCalledWith('user-123', validUser);
        });
    });
});
```

### Pattern: Object Spread for Variations

When you need variations of the same data, use object spread:

```typescript
describe('OrderService', () => {
    // ✅ Base object with common data
    const baseOrder = {
        id: 'order-123',
        customerId: 'customer-456',
        items: [
            { productId: 'prod-1', quantity: 2, price: 10 },
            { productId: 'prod-2', quantity: 1, price: 20 },
        ],
        status: 'pending',
        total: 40,
    };

    it('should calculate total for pending order', () => {
        const order = baseOrder;
        expect(service.calculateTotal(order)).toBe(40);
    });

    it('should apply discount to completed order', () => {
        const order = { ...baseOrder, status: 'completed' };
        expect(service.calculateTotal(order)).toBe(36); // 10% discount
    });

    it('should handle order with no items', () => {
        const order = { ...baseOrder, items: [], total: 0 };
        expect(service.calculateTotal(order)).toBe(0);
    });
});
```

**Note:** Only use factory functions when you need to generate unique values (IDs, timestamps) or complex logic. For simple variations, spread syntax is clearer.

### Pattern: Shared Fixtures in beforeEach

Use `beforeEach` when tests MUTATE the data (objects that change state):

```typescript
describe('ProductService', () => {
    let product: Product;

    // ❌ BAD: Recreating immutable data unnecessarily
    beforeEach(() => {
        product = { id: 'prod-1', name: 'Laptop', price: 1000 };
    });

    it('should format product name', () => {
        expect(service.formatName(product)).toBe('Laptop');
    });

    it('should calculate tax', () => {
        expect(service.calculateTax(product)).toBe(100);
    });
});

// ✅ GOOD: Extract immutable data to constant
describe('ProductService', () => {
    const product = { id: 'prod-1', name: 'Laptop', price: 1000 };

    it('should format product name', () => {
        expect(service.formatName(product)).toBe('Laptop');
    });

    it('should calculate tax', () => {
        expect(service.calculateTax(product)).toBe(100);
    });
});

// ✅ GOOD: Use beforeEach for mutable data
describe('ProductService', () => {
    let product: Product;

    beforeEach(() => {
        // Fresh instance needed because tests mutate it
        product = new Product('prod-1', 'Laptop', 1000);
    });

    it('should update product price', () => {
        service.updatePrice(product, 1200);
        expect(product.price).toBe(1200); // Mutated
    });

    it('should apply discount to product', () => {
        service.applyDiscount(product, 0.1);
        expect(product.price).toBe(900); // Mutated
    });
});
```

**Rule:** Only use `beforeEach` if tests MODIFY the object. Otherwise, use a constant.

### When to Extract vs Keep Inline

**Extract when:**
- ✅ Data appears in 2+ tests
- ✅ Data is complex (>3 properties)
- ✅ Data represents a meaningful test fixture
- ✅ Changes to data structure affect multiple tests

```typescript
// ✅ Extract: Used in multiple tests
const validCredentials = { username: 'admin', password: 'secret123' };

it('should authenticate with valid credentials', () => { /* ... */ });
it('should log access with valid credentials', () => { /* ... */ });
```

**Keep inline when:**
- ✅ Data is unique to one test
- ✅ Data is simple (1-2 properties)
- ✅ Inlining makes test more readable
- ✅ Test explicitly tests that specific data

```typescript
// ✅ Keep inline: Unique to this test
it('should reject empty username', async () => {
    await expect(service.login({ username: '', password: 'pwd' })).rejects.toThrow();
});
```

### Guidelines

1. **Extract at describe level** - For constants used across multiple tests
2. **Use beforeEach for mutable data** - Fresh instances for each test
3. **Factory functions for variations** - When data needs customization
4. **Don't over-extract** - Balance DRY with readability
5. **Name clearly** - `validUser`, `invalidEmail`, `mockApiResponse`
6. **Keep immutable** - Use `const` for shared constants

### Benefits

✅ **Single source of truth** - Update data in one place
✅ **Reduced duplication** - DRY principle applied
✅ **Fewer bugs** - No copy-paste errors
✅ **Better maintainability** - Change structure once
✅ **Clearer intent** - Naming shows data purpose

---

