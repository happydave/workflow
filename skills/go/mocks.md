---
name: go-mocks
description: Use when a Go work item uses mockery or testify mocks, generated or hand-built — expectation counts, verification, goroutine failures, matcher order
---
# Go: Mocks

Sub-file of `skills/go.md`; its writing-tests rules apply first.

- **A mock expectation asserts *at least*, never *at most*.** `EXPECT()` without `.Once()`/`.Times(n)` passes any number of calls from one upward, so a test that means "this is called once" and omits the count will also pass when the code calls it four times. Pin the count wherever the count is the behaviour under test.
- **A pinned count still needs something to enforce it.** `.Once()` catches over-calling on the spot, but under-calling is only caught when expectations are verified — by the mockery `NewFoo(t)` constructor, or by an explicit `AssertExpectations(t)`. A hand-built mock with `.Once()` and no verification passes when the call never happens.
- **A mock that fails on a non-test goroutine takes the goroutine with it.** An unexpected or over-budget call fails from wherever it was made, so a handler goroutine unwinds through its deferred calls and skips everything else (and a hand-built mock with no `Test(t)` set panics outright, taking the binary with it) — including a `done` channel closed at the bottom of the function body, which leaves the test waiting on it forever. Close such channels with `defer`.
- **mockery's loader lags the Go release.** Generation failing with `internal error: package "…" without types was imported` is a stale `golang.org/x/tools` in mockery, not a fault in your interfaces — run the generator under the previous toolchain until mockery catches up.
- **`EXPECT()` needs mockery's expecter API.** In v2 it is `with-expecter: true`; in v3 the parameter was removed and the `testify` template always generates expecters, so there is no knob to look for. Without an expecter the generated mock has `On("Method", ...)` only, and the count rules above still apply — through `.Once()` on the `On` call.
- **Synchronize before asserting on a call made off the test goroutine.** Wait for the handler to return (a done channel, a `WaitGroup`) before verifying expectations; asserting while the goroutine is still running passes or fails by scheduling, and `-count=5` will find that out.
- **testify matches expectations in declaration order.** A catch-all (`mock.Anything`) declared before a specific `mock.MatchedBy` absorbs the call and leaves the specific one unreachable. Declare the specific matcher first — and for an exactly-once assertion, do not declare a catch-all at all, since the specific expectation retires after its first call and the next one falls through. The failure reads as an **under-call in your production code** ("1 out of 2 expectation(s) were met … needs to make 1 more call(s)"), which sends you to debug the wrong side; "has been called over N times" is the different case where every matching expectation is already exhausted.
