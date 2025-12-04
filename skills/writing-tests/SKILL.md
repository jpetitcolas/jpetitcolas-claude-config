---
name: writing-tests
description: Provides testing best practices for JavaScript/TypeScript with Vitest. Use when writing tests, creating test files, fixing failing tests, mocking time or functions, or reviewing test code. Covers vi.useFakeTimers, vi.stubEnv, it.each for data variations, hard-coded assertions, avoiding duplicated tests, and focusing on behaviors over implementation details.
---

- [Time Mocking](time-mocking.md) - Never use `new Date()`, always mock with `vi.useFakeTimers()`
- [Hard-Coded Values](hard-coded-values.md) - Expected values should be explicit, not computed
- [Avoiding Duplication](avoiding-duplication.md) - Each test should verify a unique behavior
- [Focused Assertions](focused-assertions.md) - Test only what the description says
- [Using `it.each`](it-each.md) - Use for data variations instead of duplicating tests
- [Behavior Testing](behavior-testing.md) - Test what code does, not how it does it
- [Test Ordering and Flow](flow.md) - Match test order to source file method order
- [Test Organization](organization.md) - Group tests by condition with nested describe blocks
- [Method Visibility](method-visibility.md) - Test private, protected, and static methods
- [DRY Patterns](dry.md) - Extract shared data, use object spread, create fixtures
- [Mocking and Async](mocking.md) - Mock dependencies, test async functions, handle errors
- [Logs and Observability](logs.md) - Test log levels, structured logs, sensitive data redaction

