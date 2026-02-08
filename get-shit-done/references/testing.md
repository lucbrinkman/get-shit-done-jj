<overview>
Shared testing guidance for all GSD execution contexts — TDD plans, standard plans, bug fixes, and gap closures. Write unit+1 tests. Mock at the slow/external boundary. Test behavior, not implementation.

**This reference is imported by both TDD and standard execution paths.** Write it once, use everywhere.
</overview>

<test_granularity>
## Test Granularity: Unit+1 Style

Prefer **unit+1 tests** (also known as **shallow integration tests**) over pure unit tests or full integration tests.

| Level | What it means | Example |
|-------|---------------|---------|
| **Unit** | Mock direct dependencies | Mock the database in a Django test |
| **Unit+1** (preferred) | Use direct dependencies, mock what's slow/external | Call User.create() with real DB, mock 2FA service |
| **Full integration** | Real everything | Hit actual 2FA service |

**Why unit+1:**
- Tests real interactions between your code and its immediate dependencies
- Still fast (mocks cut off slow/external things)
- Catches integration bugs that pure unit tests miss
- More maintainable than mocking every direct dependency

**How deep is "+1"?** There's no fixed rule. The boundary depends on the situation — what's slow, what's external, and what interactions matter for the behavior you're testing. A unit+1 test for a service with a database is different from a unit+1 test for a parser with a formatter. Use judgment: go deep enough to test real interactions, shallow enough to stay fast.

**The heuristic:** Mock at the slow/external boundary, not at your code's direct dependencies.

### What counts as slow/external (mock these):
- Network calls (HTTP APIs, external services, webhooks)
- Third-party services (payment providers, email, SMS, OAuth)
- File system operations that hit disk
- Timers, delays, system clock
- Non-deterministic operations (random, UUIDs in assertions)

### What counts as direct dependencies (use real ones):
- Your own modules and functions
- Database queries (use test database)
- Framework utilities (routers, middleware, serializers)
- In-memory data structures
- Validation logic, business rules
</test_granularity>

<mocking_boundaries>
## Mocking Boundaries

**The iron rule:** If you're mocking something, you must know what side effects the real implementation has and whether your test depends on any of them.

### Mock at the right level

```
YOUR CODE → Direct dependency → Slow/external thing
              ↑ use real           ↑ mock here
```

**Good:** Mock the HTTP client, not the service class that uses it
**Bad:** Mock the service class to avoid its HTTP calls

### When mocks become too complex

Warning signs:
- Mock setup longer than test logic
- Mocking everything to make test pass
- Mock missing methods the real component has
- Test breaks when mock changes

**Diagnosis:** You're mocking at the wrong level. Mock further from your code, at the slow/external boundary.

### Complete mocks

When creating mock responses (API responses, database results), mirror the **complete** real data structure, not just the fields your immediate test uses. Downstream code may depend on fields you omitted.

```typescript
// BAD: Partial mock
const mockUser = { id: '123', name: 'Alice' };
// Missing: email, createdAt, role — downstream code may need these

// GOOD: Complete mock
const mockUser = { id: '123', name: 'Alice', email: 'alice@test.com', createdAt: new Date(), role: 'user' };
```
</mocking_boundaries>

<test_quality>
## Test Quality

**Test behavior, not implementation:**
- Good: "returns formatted date string"
- Bad: "calls formatDate helper with correct params"
- Tests should survive refactors

**One concept per test:**
- Good: Separate tests for valid input, empty input, malformed input
- Bad: Single test checking all edge cases with multiple assertions

**Descriptive names:**
- Good: "should reject empty email", "returns null for invalid ID"
- Bad: "test1", "handles error", "works correctly"

**No implementation details:**
- Good: Test public API, observable behavior
- Bad: Mock internals, test private methods, assert on internal state
</test_quality>

<when_to_test>
## When to Write Tests

**TDD plans (`type: tdd`):** Test-first. RED → GREEN → REFACTOR. See `references/tdd.md`.

**Standard plans (`type: execute`):** Write tests for any task that has testable behavior — validation, business logic, API contracts, data transformations. The task may not be a TDD plan, but that doesn't mean "no tests." If the task's `<action>` describes behavior with inputs and outputs, write a test for it.

**Bug fixes (deviation rules):** Always write a failing test reproducing the bug before fixing it. The test proves the fix works and prevents regression.

**Gap closures:** Follow the same rules as the original plan type.

### The bar for skipping tests

You can skip tests only when there is **genuinely no testable behavior:**
- Purely visual changes (CSS, layout, styling)
- Configuration file changes
- Glue code connecting existing *tested* components
- Framework-provided CRUD with no custom logic

"It's simple" is not a reason to skip tests. Simple logic still has contracts.
</when_to_test>
