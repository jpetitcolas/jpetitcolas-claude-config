# Testing Patterns - Test Ordering and Flow

Patterns for organizing tests to match source code structure and logic flow.

## Table of Contents

1. [Match Source File Method Order](#match-source-file-method-order)
2. [Match Test Flow to Source Code Flow](#match-test-flow-to-source-code-flow)

---

## Match Source File Method Order

**Principle:** Test methods should appear in the same order as the methods in the source file being tested.

**Why:**
- **Easy navigation** - Jump between source and test files seamlessly
- **Better maintainability** - Changes to source order naturally reflect in tests
- **Clearer structure** - Test file mirrors source file organization
- **Reduces confusion** - No hunting for where a method is tested

### Example: Proper Test Ordering

```typescript
// Source: UserService.ts
export class UserService {
    constructor(private db: Database, private emailer: EmailService) {}

    async createUser(userData: UserData): Promise<User> {
        // Implementation
    }

    async updateUser(userId: string, updates: Partial<UserData>): Promise<User> {
        // Implementation
    }

    async deleteUser(userId: string): Promise<void> {
        // Implementation
    }

    async findUserById(userId: string): Promise<User | null> {
        // Implementation
    }

    async listUsers(filters?: UserFilters): Promise<User[]> {
        // Implementation
    }
}

// Test: UserService.test.ts
describe('UserService', () => {
    let service: UserService;
    let mockDb: Database;
    let mockEmailer: EmailService;

    beforeEach(() => {
        mockDb = createMockDatabase();
        mockEmailer = createMockEmailer();
        service = new UserService(mockDb, mockEmailer);
    });

    // ✅ Tests appear in the SAME ORDER as source file methods
    describe('createUser', () => {
        it('should create user with valid data', async () => {
            // Test implementation
        });

        it('should send welcome email after creation', async () => {
            // Test implementation
        });

        it('should throw error when email already exists', async () => {
            // Test implementation
        });
    });

    describe('updateUser', () => {
        it('should update user fields', async () => {
            // Test implementation
        });

        it('should throw error when user not found', async () => {
            // Test implementation
        });
    });

    describe('deleteUser', () => {
        it('should delete user by id', async () => {
            // Test implementation
        });

        it('should throw error when user not found', async () => {
            // Test implementation
        });
    });

    describe('findUserById', () => {
        it('should return user when found', async () => {
            // Test implementation
        });

        it('should return null when not found', async () => {
            // Test implementation
        });
    });

    describe('listUsers', () => {
        it('should return all users when no filters', async () => {
            // Test implementation
        });

        it('should filter users by status', async () => {
            // Test implementation
        });
    });
});
```

### Bad Example: Random Test Ordering

```typescript
// ❌ Bad: Random order makes navigation difficult
describe('UserService', () => {
    // Tests are not in source file order
    describe('listUsers', () => { /* ... */ });

    describe('deleteUser', () => { /* ... */ });

    describe('findUserById', () => { /* ... */ });

    describe('createUser', () => { /* ... */ });

    describe('updateUser', () => { /* ... */ });

    // Problem: Have to search through test file to find tests
    // Problem: Doesn't match mental model from reading source
    // Problem: Harder to spot missing tests
});
```

---

## Match Test Flow to Source Code Flow

**Principle:** Order tests to mirror the logical flow of the source code (conditions, branches, steps).

**Why:**
- **Easy to verify coverage** - Walk through code and tests in parallel
- **Better readability** - Tests tell the story of how the code works
- **Spot missing tests** - Gaps in coverage immediately visible
- **Easier maintenance** - Changes to flow naturally reflected in test order

#### Example: Function with Conditional Logic

```typescript
// Source: UserService.ts
async createUser(userData: UserData): Promise<User> {
    // Step 1: Validate input
    if (!userData.email) {
        throw new ValidationError('Email is required');
    }

    if (!this.isValidEmail(userData.email)) {
        throw new ValidationError('Invalid email format');
    }

    // Step 2: Check for existing user
    const existing = await this.userRepository.findByEmail(userData.email);
    if (existing) {
        throw new ConflictError('User already exists');
    }

    // Step 3: Create user
    const user = await this.userRepository.create(userData);

    // Step 4: Send welcome email
    await this.emailService.sendWelcome(user.email);

    return user;
}

// Test: UserService.test.ts
describe('UserService', () => {
    describe('createUser', () => {
        // ✅ Tests follow source code flow

        // Step 1: Validation tests (matches source step 1)
        describe('validation', () => {
            it('should throw when email is missing', async () => {
                await expect(
                    service.createUser({ name: 'Alice' })
                ).rejects.toThrow('Email is required');
            });

            it('should throw when email format is invalid', async () => {
                await expect(
                    service.createUser({ email: 'invalid' })
                ).rejects.toThrow('Invalid email format');
            });
        });

        // Step 2: Existing user check (matches source step 2)
        describe('when user already exists', () => {
            beforeEach(() => {
                mockUserRepository.findByEmail.mockResolvedValue({
                    id: 'existing-user',
                    email: 'alice@example.com',
                });
            });

            it('should throw ConflictError', async () => {
                await expect(
                    service.createUser({ email: 'alice@example.com' })
                ).rejects.toThrow('User already exists');
            });
        });

        // Step 3 & 4: Happy path (matches source steps 3-4)
        describe('when user does not exist', () => {
            beforeEach(() => {
                mockUserRepository.findByEmail.mockResolvedValue(null);
                mockUserRepository.create.mockResolvedValue({
                    id: 'user-123',
                    email: 'alice@example.com',
                });
            });

            it('should create user in repository', async () => {
                await service.createUser({ email: 'alice@example.com' });

                expect(mockUserRepository.create).toHaveBeenCalledWith({
                    email: 'alice@example.com',
                });
            });

            it('should send welcome email', async () => {
                await service.createUser({ email: 'alice@example.com' });

                expect(mockEmailService.sendWelcome).toHaveBeenCalledWith(
                    'alice@example.com'
                );
            });

            it('should return created user', async () => {
                const result = await service.createUser({
                    email: 'alice@example.com',
                });

                expect(result).toEqual({
                    id: 'user-123',
                    email: 'alice@example.com',
                });
            });
        });
    });
});
```

#### Example: Matching Conditional Branches

```typescript
// Source: PaymentProcessor.ts
async processPayment(payment: Payment): Promise<ProcessResult> {
    // Branch 1: Check payment amount
    if (payment.amount <= 0) {
        this.logger.warn({ event: 'invalid_amount', amount: payment.amount });
        throw new ValidationError('Amount must be positive');
    }

    // Branch 2: Check user balance
    const balance = await this.getBalance(payment.userId);
    if (balance < payment.amount) {
        this.logger.info({ event: 'insufficient_funds', userId: payment.userId });
        return { status: 'declined', reason: 'insufficient_funds' };
    }

    // Branch 3: Process payment
    try {
        const result = await this.gateway.charge(payment);
        this.logger.info({ event: 'payment_success', paymentId: result.id });
        return { status: 'success', transactionId: result.id };
    } catch (error) {
        this.logger.error({ event: 'payment_failed', error: error.message });
        return { status: 'failed', reason: 'gateway_error' };
    }
}

// Test: PaymentProcessor.test.ts
describe('PaymentProcessor', () => {
    describe('processPayment', () => {
        // ✅ Tests follow source branches in order

        // Branch 1: Invalid amount (matches source branch 1)
        describe('when amount is invalid', () => {
            it('should throw ValidationError for zero amount', async () => {
                const payment = { amount: 0, userId: 'user-123' };

                await expect(processor.processPayment(payment)).rejects.toThrow(
                    'Amount must be positive'
                );
            });

            it('should throw ValidationError for negative amount', async () => {
                const payment = { amount: -10, userId: 'user-123' };

                await expect(processor.processPayment(payment)).rejects.toThrow(
                    'Amount must be positive'
                );
            });

            it('should log warning for invalid amount', async () => {
                const payment = { amount: 0, userId: 'user-123' };

                await expect(processor.processPayment(payment)).rejects.toThrow();

                expect(mockLogger.warn).toHaveBeenCalledWith(
                    expect.objectContaining({ event: 'invalid_amount' })
                );
            });
        });

        // Branch 2: Insufficient funds (matches source branch 2)
        describe('when balance is insufficient', () => {
            beforeEach(() => {
                mockGetBalance.mockResolvedValue(50);
            });

            it('should return declined status', async () => {
                const payment = { amount: 100, userId: 'user-123' };

                const result = await processor.processPayment(payment);

                expect(result).toEqual({
                    status: 'declined',
                    reason: 'insufficient_funds',
                });
            });

            it('should log insufficient funds', async () => {
                const payment = { amount: 100, userId: 'user-123' };

                await processor.processPayment(payment);

                expect(mockLogger.info).toHaveBeenCalledWith(
                    expect.objectContaining({ event: 'insufficient_funds' })
                );
            });
        });

        // Branch 3a: Successful payment (matches source branch 3 success path)
        describe('when payment succeeds', () => {
            beforeEach(() => {
                mockGetBalance.mockResolvedValue(200);
                mockGateway.charge.mockResolvedValue({ id: 'txn-456' });
            });

            it('should return success with transaction ID', async () => {
                const payment = { amount: 100, userId: 'user-123' };

                const result = await processor.processPayment(payment);

                expect(result).toEqual({
                    status: 'success',
                    transactionId: 'txn-456',
                });
            });

            it('should log payment success', async () => {
                const payment = { amount: 100, userId: 'user-123' };

                await processor.processPayment(payment);

                expect(mockLogger.info).toHaveBeenCalledWith(
                    expect.objectContaining({ event: 'payment_success' })
                );
            });
        });

        // Branch 3b: Gateway failure (matches source branch 3 error path)
        describe('when gateway fails', () => {
            beforeEach(() => {
                mockGetBalance.mockResolvedValue(200);
                mockGateway.charge.mockRejectedValue(new Error('Network timeout'));
            });

            it('should return failed status', async () => {
                const payment = { amount: 100, userId: 'user-123' };

                const result = await processor.processPayment(payment);

                expect(result).toEqual({
                    status: 'failed',
                    reason: 'gateway_error',
                });
            });

            it('should log payment failure', async () => {
                const payment = { amount: 100, userId: 'user-123' };

                await processor.processPayment(payment);

                expect(mockLogger.error).toHaveBeenCalledWith(
                    expect.objectContaining({ event: 'payment_failed' })
                );
            });
        });
    });
});
```

#### Benefits of Matching Flow

**For readers:**
- ✅ Open source file and test file side by side
- ✅ Walk through code and see corresponding test
- ✅ Verify each branch has test coverage
- ✅ Understand code behavior from test structure

**For maintainers:**
- ✅ Spot missing test cases for new branches
- ✅ Know exactly where to add tests for new conditions
- ✅ Refactor code and tests together
- ✅ Delete tests when removing branches

#### Guidelines

1. **Follow source order** - Tests appear in same order as code flow
2. **Match conditionals** - Each if/else branch gets its own test group
3. **Match steps** - Sequential operations tested sequentially
4. **Match error paths** - Test validation in order it appears
5. **Use describe blocks** - Group tests by branch/condition
6. **Name by condition** - Describe blocks named after source conditions

#### Anti-Pattern: Random Test Order

```typescript
// ❌ Bad: Tests in random order, not matching source flow
describe('processPayment', () => {
    it('should log payment success', async () => { /* ... */ });
    it('should throw for negative amount', async () => { /* ... */ });
    it('should return failed status when gateway fails', async () => { /* ... */ });
    it('should return declined for insufficient funds', async () => { /* ... */ });
    it('should throw for zero amount', async () => { /* ... */ });

    // Problem: Hard to match tests to source code
    // Problem: Can't verify all branches covered
    // Problem: No clear organization
});
```

---

## Guidelines

1. **Match source file order** - Test describes appear in same order as methods in source
2. **Match code flow within methods** - Test conditions/branches in execution order
3. **Use describe blocks** - Group tests by method and by condition
4. **Name by condition** - Describe blocks named after source conditions
5. **Maintain consistency** - Same pattern across all test files

### Benefits

✅ **Easy to find tests** - Know exactly where tests are for any method
✅ **Spot missing tests** - Gaps in test coverage are obvious
✅ **Better code reviews** - Reviewers can navigate easily
✅ **Reduced cognitive load** - Predictable structure
✅ **Easier refactoring** - Clear 1:1 mapping between source and tests

---

