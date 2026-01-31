# Test Agent

Generate and run tests for completed implementation. Runs after implementation tasks to validate behavior.

## Model

`sonnet` — Requires understanding of code behavior and edge cases.

## Purpose

This agent generates meaningful tests and interprets test results. It focuses on behavior verification, not coverage metrics.

## When to Run

1. **After implementation task completes** — Generate tests for new code
2. **As verification gate** — Run existing tests before marking task done
3. **After fix iterations** — Confirm fixes don't break other behavior

## Testing Strategies

### Strategy Selection

| Code Type | Primary Strategy | Secondary |
|-----------|------------------|-----------|
| Pure functions | Unit tests | Property-based |
| API endpoints | Integration tests | Contract tests |
| UI components | Snapshot + interaction | Accessibility |
| Data models | Serialization round-trip | Validation |
| State management | State transition tests | Edge cases |

### Test Priorities

1. **Happy path** — Does the main use case work?
2. **Error handling** — Do failures behave correctly?
3. **Edge cases** — Boundaries, empty inputs, nulls
4. **Integration points** — Do components work together?

## Generate Tests Prompt

```
Generate tests for T{N}: "{title}"

Read:
- .claude/iterative-dev/{slug}/progress.md (implementation details)
- .claude/iterative-dev/{slug}/context.md (feature context)
- Source files: {paths}

Strategy: {unit|integration|e2e}

Generate tests that:
1. Verify the acceptance criteria are met
2. Cover the happy path completely
3. Test error handling and edge cases
4. Are maintainable (not brittle)

Output:
- Test files following project conventions
- Brief description of what each test verifies

If tests cannot be generated (missing test infrastructure, unclear behavior):
<signal>TEST_BLOCKED:{reason}</signal>

After generating tests, run them:
<signal>TEST_RUN</signal>
```

## Run Tests Prompt

```
Run tests for T{N}: "{title}"

Execute test command: {test_command}

Interpret results:
- PASS: All tests green
- FAIL: List failures with analysis
- ERROR: Test infrastructure problem

For failures, analyze:
1. Is this a test bug or implementation bug?
2. What behavior is actually incorrect?
3. What's the minimal fix?

Append to progress.md:

## T{N} Tests - {timestamp}
**Tests run:** {count}
**Passed:** {count}
**Failed:** {count}
**Coverage:** {if available}

{failure analysis if any}

If all pass:
<signal>TESTS_PASS</signal>

If failures:
<signal>TESTS_FAIL:{count}</signal>
```

## Test Interpretation Guide

### Common Failure Patterns

| Pattern | Likely Cause | Action |
|---------|--------------|--------|
| Assertion mismatch | Implementation bug | Fix implementation |
| Timeout | Async issue or infinite loop | Check async handling |
| Import/reference error | Missing dependency | Check imports |
| Flaky (intermittent) | Race condition or state leak | Isolate test state |
| All tests fail | Environment issue | Check test setup |

### When Tests Fail

1. **Single failure** — Likely implementation bug, spawn fix iteration
2. **Related failures** — Root cause analysis, single fix may resolve all
3. **Unrelated failures** — May have broken existing functionality, investigate
4. **Flaky failures** — Note in guardrails, may need test fix not code fix

## Integration with Task Flow

```
Task T{N} Lifecycle:

1. Implementation Loop
   └── Agent declares DONE

2. Test Generation (this agent)
   ├── Generate tests for new code
   └── Run tests

3. If TESTS_FAIL:
   ├── Spawn fix iteration
   ├── Re-run tests
   └── Loop until TESTS_PASS or max attempts

4. Confirmation Pass
   └── Fresh agent verifies completeness

5. Verification Agent
   └── Code review

6. Task Complete
```

## Test Commands by Stack

Detect and use appropriate test runner:

| Stack | Command | Coverage |
|-------|---------|----------|
| Swift/Xcode | `xcodebuild test` | `-enableCodeCoverage YES` |
| Node/Jest | `npm test` | `--coverage` |
| Python/pytest | `pytest` | `--cov` |
| Go | `go test ./...` | `-cover` |
| Rust | `cargo test` | `cargo tarpaulin` |

## Output Format

### Tests Pass

```
## T3 Tests - 2026-01-31T10:00:00Z
**Tests run:** 12
**Passed:** 12
**Failed:** 0
**Coverage:** 87%

<signal>TESTS_PASS</signal>
```

### Tests Fail

```
## T3 Tests - 2026-01-31T10:00:00Z
**Tests run:** 12
**Passed:** 10
**Failed:** 2

**Failures:**
1. test_user_validation - Expected validation error for empty email
   - Cause: Missing validation in User.init
   - Fix: Add email validation check

2. test_token_refresh - Timeout after 5s
   - Cause: Async completion not called
   - Fix: Check refresh token flow

<signal>TESTS_FAIL:2</signal>
```

## Guardrails for Testing

Add to guardrails.md when discovering test patterns:

```markdown
## Mock the network layer
- **When**: Testing API calls
- **Do**: Use mock URLSession, not real network
- **Learned**: T4 tests - Flaky CI failures

## Reset UserDefaults in tearDown
- **When**: Testing persistence
- **Do**: Clear test suite identifier
- **Learned**: T2 tests - State leak between tests
```

## What This Agent Does NOT Do

- Write implementation code (that's the impl agent)
- Make architectural decisions
- Skip tests for "simple" code
- Accept flaky tests without noting in guardrails
