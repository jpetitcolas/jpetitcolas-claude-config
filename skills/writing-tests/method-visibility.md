# Testing Patterns - Method Visibility

Patterns for testing private, protected, and static methods.

## Table of Contents

1. [Private Methods](#private-methods)
2. [Protected Methods](#protected-methods)
3. [Static Methods](#static-methods)

---

## Private Methods

**Principle:** Private methods should ONLY be tested through public methods. Never test them directly.

**Why:** Private methods are implementation details that can change without affecting external behavior.

```typescript
// Source: PaymentService.ts
export class PaymentService {
    async processPayment(amount: number): Promise<Result> {
        const validated = this.validateAmount(amount); // private
        const formatted = this.formatAmount(validated); // private
        return this.charge(formatted); // private
    }

    private validateAmount(amount: number): number { /* ... */ }
    private formatAmount(amount: number): number { /* ... */ }
    private charge(amount: number): Promise<Result> { /* ... */ }
}

// Test: PaymentService.test.ts
describe('PaymentService', () => {
    // ✅ Good: Test private methods through public API
    describe('processPayment', () => {
        it('should process valid payment', async () => {
            // Tests validateAmount, formatAmount, and charge indirectly
        });

        it('should reject negative amount', async () => {
            // Tests validateAmount indirectly
        });

        it('should format amount correctly', async () => {
            // Tests formatAmount indirectly
        });
    });

    // ❌ Bad: Don't create separate describe blocks for private methods
    // describe('validateAmount', () => { /* ... */ });
    // describe('formatAmount', () => { /* ... */ });
});
```

---

## Protected Methods

**Principle:** Protected methods CAN be tested independently by creating a test subclass that exposes them.

**Why:** Protected methods are part of the inheritance contract and may contain complex logic worth testing directly.

```typescript
// Source: BaseService.ts
export abstract class BaseService {
    async processData(data: any): Promise<Result> {
        const validated = this.validateData(data);
        return this.transform(validated);
    }

    protected validateData(data: any): any {
        // Complex validation logic
        if (!data.id) throw new Error('ID required');
        if (!data.timestamp) throw new Error('Timestamp required');
        if (data.timestamp < Date.now() - 3600000) {
            throw new Error('Data too old');
        }
        return data;
    }

    protected abstract transform(data: any): Promise<Result>;
}

// Test: BaseService.test.ts
describe('BaseService', () => {
    // ✅ Create test subclass to expose protected methods
    class TestService extends BaseService {
        protected transform(data: any): Promise<Result> {
            return Promise.resolve({ success: true, data });
        }

        // ✅ Expose protected method for direct testing
        public testValidateData(data: any): any {
            return this.validateData(data);
        }
    }

    let service: TestService;
    const FIXED_TIME = new Date('2025-01-10T12:00:00Z').getTime();

    beforeEach(() => {
        vi.useFakeTimers();
        vi.setSystemTime(FIXED_TIME);
        service = new TestService();
    });

    afterEach(() => {
        vi.useRealTimers();
    });

    // ✅ Test protected method independently
    describe('validateData', () => {
        it('should throw when id is missing', () => {
            expect(() => service.testValidateData({ timestamp: FIXED_TIME }))
                .toThrow('ID required');
        });

        it('should throw when data is too old', () => {
            const oldTimestamp = FIXED_TIME - 7200000;

            expect(() => service.testValidateData({
                id: '123',
                timestamp: oldTimestamp
            })).toThrow('Data too old');
        });

        it('should return validated data when valid', () => {
            const data = { id: '123', timestamp: FIXED_TIME };
            const result = service.testValidateData(data);

            expect(result).toEqual(data);
        });
    });
});
```

**When to test protected methods directly:**
- Complex logic that's hard to cover through public API
- Multiple edge cases that need thorough testing
- Reusable logic across multiple subclasses

---

## Static Methods

**Principle:** Static methods should be tested in their own section, typically at the end of the test file.

**Why:** Static methods don't depend on instance state, so group them together for clarity.

```typescript
// Source: StringUtils.ts
export class StringUtils {
    static capitalize(str: string): string { /* ... */ }
    static truncate(str: string, length: number): string { /* ... */ }
    static slugify(str: string): string { /* ... */ }
}

// Test: StringUtils.test.ts
describe('StringUtils', () => {
    describe('capitalize', () => {
        it('should capitalize first letter', () => { /* ... */ });
    });

    describe('truncate', () => {
        it('should truncate long strings', () => { /* ... */ });
    });

    describe('slugify', () => {
        it('should convert to slug format', () => { /* ... */ });
    });
});
```

**For mixed instance and static methods:**

```typescript
// Source: UserService.ts
export class UserService {
    // Instance methods
    async createUser(data: UserData): Promise<User> { /* ... */ }
    async updateUser(id: string, data: UserData): Promise<User> { /* ... */ }

    // Static methods
    static validateEmail(email: string): boolean { /* ... */ }
    static formatName(name: string): string { /* ... */ }
}

// Test: UserService.test.ts
describe('UserService', () => {
    // ✅ Instance methods first (in source order)
    describe('createUser', () => { /* ... */ });
    describe('updateUser', () => { /* ... */ });

    // ✅ Static methods grouped at the end
    describe('static methods', () => {
        describe('validateEmail', () => { /* ... */ });
        describe('formatName', () => { /* ... */ });
    });
});
```

---

## Guidelines

1. **Private methods** - Test only through public API
2. **Protected methods** - Create test subclass to expose them when needed
3. **Static methods** - Group together, typically at end of test file
4. **Test public interfaces** - Focus on behavior, not implementation
5. **Maintain consistency** - Same pattern across all test files

## Benefits

✅ **Clear boundaries** - Understand what to test directly vs indirectly
✅ **Better encapsulation** - Private implementation can change freely
✅ **Protected testing** - Can test inheritance contract thoroughly
✅ **Static clarity** - Static methods grouped for easy navigation
✅ **Maintainable tests** - Tests survive refactoring

---
