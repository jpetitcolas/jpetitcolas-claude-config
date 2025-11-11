# Testing Patterns - Test Organization

Patterns for organizing test suites using nested describe blocks. Part of the [Testing Guidelines](../../SKILL.md) skill.

## Table of Contents

1. [Group Tests by Shared Condition](#group-tests-by-shared-condition-with-nested-describe)

---

## Group Tests by Shared Condition with Nested `describe`

**Principle:** When multiple tests share the same setup/condition, group them under a nested `describe` block.

**Code Smell:** Duplicated setup across multiple `it` blocks signals missing `describe`.

**Why:**
- **DRY principle** - Setup defined once in `beforeEach`
- **Clear context** - Test intent immediately obvious
- **Better organization** - Related tests grouped together
- **Easier to read** - Condition explicit in `describe` name

#### Example: Duplicate Lifecycle Detection

```typescript
// ❌ Bad: Repeated condition/setup across tests
describe('LifecycleController', () => {
    describe('handleStarted', () => {
        it('should throw if lifecycle already started', async () => {
            // Repeated setup
            const existingMetadata = {
                environment_id: 'env-123',
                environment_name: 'preview-pr-42',
                application_name: 'hermes-api',
                deployment_id: 'deploy-123',
                timestamp: 1698765000,
            };

            vi.mocked(mockTrackingService.getMetadata).mockResolvedValue(
                existingMetadata
            );

            const payload = {
                environment_id: 'env-123',
                environment_name: 'preview-pr-42',
                application_name: 'hermes-api',
                deployment_id: 'deploy-123',
                timestamp: 1698765432,
            };

            await expect(controller.handleStarted(payload)).rejects.toThrow(
                BadRequestException
            );
        });

        it('should log warning for duplicate lifecycle start', async () => {
            // Same setup repeated!
            const existingMetadata = {
                environment_id: 'env-123',
                environment_name: 'preview-pr-42',
                application_name: 'hermes-api',
                deployment_id: 'deploy-123',
                timestamp: 1698765000,
            };

            vi.mocked(mockTrackingService.getMetadata).mockResolvedValue(
                existingMetadata
            );

            const payload = {
                environment_id: 'env-123',
                environment_name: 'preview-pr-42',
                application_name: 'hermes-api',
                deployment_id: 'deploy-123',
                timestamp: 1698765432,
            };

            await expect(controller.handleStarted(payload)).rejects.toThrow();

            expect(mockLogger.warn).toHaveBeenCalledWith({
                event: 'webhook_deployment_duplicate',
                message: 'deployment start already recorded',
                existing_metadata: existingMetadata,
            });
        });
    });
});

// ✅ Good: Group by shared condition
describe('LifecycleController', () => {
    describe('handleStarted', () => {
        // Nested describe for shared condition
        describe('when lifecycle already started', () => {
            const existingMetadata = {
                environment_id: 'env-123',
                environment_name: 'preview-pr-42',
                application_name: 'hermes-api',
                deployment_id: 'deploy-123',
                timestamp: 1698765000,
            };

            const payload = {
                environment_id: 'env-123',
                environment_name: 'preview-pr-42',
                application_name: 'hermes-api',
                deployment_id: 'deploy-123',
                timestamp: 1698765432,
            };

            beforeEach(() => {
                vi.mocked(mockTrackingService.getMetadata).mockResolvedValue(
                    existingMetadata
                );
            });

            it('should throw BadRequestException', async () => {
                await expect(controller.handleStarted(payload)).rejects.toThrow(
                    BadRequestException
                );
            });

            it('should log warning with existing metadata', async () => {
                await expect(controller.handleStarted(payload)).rejects.toThrow();

                expect(mockLogger.warn).toHaveBeenCalledWith({
                    event: 'webhook_deployment_duplicate',
                    message: 'deployment start already recorded',
                    existing_metadata: existingMetadata,
                });
            });
        });

        describe('when lifecycle not started', () => {
            beforeEach(() => {
                vi.mocked(mockTrackingService.getMetadata).mockResolvedValue(null);
            });

            it('should save metadata', async () => {
                const payload = { /* ... */ };
                await controller.handleStarted(payload);
                expect(mockTrackingService.saveMetadata).toHaveBeenCalled();
            });

            it('should return success response', async () => {
                const payload = { /* ... */ };
                const result = await controller.handleStarted(payload);
                expect(result.status).toBe('success');
            });
        });
    });
});
```

#### Example: User Authentication States

```typescript
describe('AuthService', () => {
    describe('login', () => {
        // Group tests by user state
        describe('when user is active', () => {
            beforeEach(() => {
                mockUserRepository.findByEmail.mockResolvedValue({
                    id: 'user-123',
                    email: 'alice@example.com',
                    status: 'active',
                    passwordHash: 'hashed-password',
                });
            });

            it('should authenticate with valid password', async () => {
                const result = await service.login('alice@example.com', 'correct-pwd');
                expect(result.token).toBeDefined();
            });

            it('should update last login timestamp', async () => {
                await service.login('alice@example.com', 'correct-pwd');
                expect(mockUserRepository.updateLastLogin).toHaveBeenCalled();
            });

            it('should log successful login', async () => {
                await service.login('alice@example.com', 'correct-pwd');
                expect(mockLogger.info).toHaveBeenCalledWith(
                    expect.objectContaining({ event: 'login_success' })
                );
            });
        });

        describe('when user is suspended', () => {
            beforeEach(() => {
                mockUserRepository.findByEmail.mockResolvedValue({
                    id: 'user-123',
                    email: 'alice@example.com',
                    status: 'suspended',
                    passwordHash: 'hashed-password',
                });
            });

            it('should reject login', async () => {
                await expect(
                    service.login('alice@example.com', 'correct-pwd')
                ).rejects.toThrow('Account suspended');
            });

            it('should log suspension attempt', async () => {
                await expect(
                    service.login('alice@example.com', 'correct-pwd')
                ).rejects.toThrow();

                expect(mockLogger.warn).toHaveBeenCalledWith(
                    expect.objectContaining({ event: 'login_suspended_account' })
                );
            });
        });
    });
});
```

#### When to Create Nested `describe`

**Create nested `describe` when:**
- ✅ 2+ tests share the same precondition/setup
- ✅ Tests represent different aspects of the same scenario
- ✅ Setup code is duplicated across tests
- ✅ Tests logically belong together (same context)

```typescript
// ✅ Good: Multiple tests, same condition
describe('when user has admin role', () => {
    beforeEach(() => { /* setup admin user */ });
    it('should allow user deletion', () => { /* ... */ });
    it('should allow role changes', () => { /* ... */ });
    it('should log admin actions', () => { /* ... */ });
});
```

**Keep flat when:**
- ✅ Only one test for a condition
- ✅ Tests have different setups
- ✅ No shared context

```typescript
// ✅ Good: Single test, no need for nested describe
it('should handle empty database', async () => {
    mockDb.findAll.mockResolvedValue([]);
    const result = await service.getAll();
    expect(result).toEqual([]);
});
```

#### Naming Nested `describe` Blocks

Use natural language describing the condition:

```typescript
// ✅ Good names - describe the condition/state
describe('when user is authenticated', () => { /* ... */ });
describe('when payment fails', () => { /* ... */ });
describe('when cache is empty', () => { /* ... */ });
describe('with invalid token', () => { /* ... */ });
describe('given existing order', () => { /* ... */ });

// ❌ Bad names - not descriptive
describe('test case 1', () => { /* ... */ });
describe('authenticated', () => { /* ... */ }); // Incomplete
describe('error', () => { /* ... */ }); // Too vague
```

#### Benefits

✅ **Eliminates duplication** - Setup in one `beforeEach`
✅ **Clear context** - Condition explicit in `describe` name
✅ **Better organization** - Related tests grouped
✅ **Easier navigation** - Find tests by condition
✅ **Self-documenting** - Test structure tells the story

---

