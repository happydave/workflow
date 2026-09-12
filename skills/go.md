---
name: go
description: Go language conventions, tooling, and testing rules
---
# Go Language Guidelines

## Purpose
These guidelines are intended to ensure consistent, idiomatic Go code.

The rules focus on unambiguous setup and tooling behavior so AI-generated code remains correct and maintainable without unnecessary decisions being forced.

## Core Principles
- Follow official Go idioms (Effective Go, Go Proverbs) unless explicitly overridden here.
- Prefer simplicity, explicitness, and standard practices.
- Prefer writing base library code over importing libraries for trivial functions.
- Only constrain what is necessary for correctness or project consistency.
- Explicitly grant freedom on non-critical choices.

## Testing
- *NEVER* use `-short` with `go test`. There is no plan-level override for this.
- *NEVER* run `go build` to test; use `go test` (or `go run` if you want to interact with a running instance).
- Always run `go test ./...` before making changes to verify the state of the project.
- Always run `go test ./...` after all changes are made for final verification.
- Pass `-timeout` on any run that can block: the default is generous enough that a hung package looks like a slow one, and the timeout is what turns it into a goroutine dump.
- **Run the documented gate, not a faster decomposition of it.** If the project documents one whole-module command as its gate, run that command. Splitting it into parallel halves changes what is tested — inter-package contention is part of what the combined run exercises, and a split has hidden a real regression for a whole session. If the combined command is too slow, record that fact; it is not a licence to substitute the halves.
- `TestMain` must live in a `_test.go` file. A `TestMain` in a regular file compiles without complaint, passes `go vet`, and never runs — no gate in this document catches it.
- **A benchmark's parallelism is part of its claim.** `-cpu` and `b.RunParallel` set the concurrency
  the result is *about*. Go weights mutex profiles by the number of blocked goroutines precisely
  because a lock with 100 waiters dominates one with 1 — so a change to lock scope measured at tens
  of goroutines says nothing about thousands, and "neutral in the benchmark" has shipped a change
  that stopped a system's connect path completing at all. Measure at the concurrency the system
  reaches, or record that you did not.
- **Concurrent test harnesses need production-grade discipline.** Collectors, fakes, and recorders shared between the code under test and the test itself must be locked like any shared state. Run a new concurrent package with `-race -count=5` before calling it stable — a single green `-race` run is one schedule, not evidence.
- **Force the contention you want to detect.** The detector reports only interleavings a test actually produces, so a concurrent path needs a test that drives many goroutines at it — a happy-path call proves nothing about a lock you forgot. This is the design half of the rule above; `-count=5` is the repetition half.
- **A handler that blocks for a connection's lifetime** (WebSocket, gRPC stream, long-poll) takes its lifecycle coverage from the protocol, not from the work item's acceptance criteria. Minimum: idle/keepalive timeout (inject short timings through test-only fields rather than waiting real durations), server-initiated graceful close, concurrent-writer safety under `-race`, and the error/close-code paths. Injected intervals stay at tens of milliseconds, not microseconds, and assertions read events — a channel receive, a call count — never elapsed time. A timer still belongs in the `select`, as the arm that turns a hang into a named failure; what it must never be is the thing the test asserts on.
- **A test that blocks a production goroutine must be able to release it from both paths.** Bind the release first — `unblock := sync.OnceFunc(func() { close(release) })` — then `t.Cleanup(unblock)` *and* call `unblock()` where the assertion needs the goroutine to move. Ordering has a precondition most tests get wrong: every `defer` in the test body runs **before** any `t.Cleanup`, so a blocking teardown written as `defer srv.Close()` or `defer wg.Wait()` can never be outrun. Register the blocking teardown with `t.Cleanup` too, then register the release after it — cleanups are LIFO, so "after" means it runs first. `OnceFunc` makes the extra registration free. Releasing only at teardown hangs the package run when an assertion fails; closing the channel in both places panics. A hung package prints nothing, which is strictly less diagnosable than a failure — pass `-timeout` so a hang becomes a goroutine dump.
- **A mock expectation asserts *at least*, never *at most*.** `EXPECT()` without `.Once()`/`.Times(n)` passes any number of calls from one upward, so a test that means "this is called once" and omits the count will also pass when the code calls it four times. Pin the count wherever the count is the behaviour under test.
- **A pinned count still needs something to enforce it.** `.Once()` catches over-calling on the spot, but under-calling is only caught when expectations are verified — by the mockery `NewFoo(t)` constructor, or by an explicit `AssertExpectations(t)`. A hand-built mock with `.Once()` and no verification passes when the call never happens.
- **A mock that fails on a non-test goroutine takes the goroutine with it.** An unexpected or over-budget call fails from wherever it was made, so a handler goroutine unwinds through its deferred calls and skips everything else (and a hand-built mock with no `Test(t)` set panics outright, taking the binary with it) — including a `done` channel closed at the bottom of the function body, which leaves the test waiting on it forever. Close such channels with `defer`.
- **`EXPECT()` needs mockery's expecter API.** In v2 it is `with-expecter: true`; in v3 the parameter was removed and the `testify` template always generates expecters, so there is no knob to look for. Without an expecter the generated mock has `On("Method", ...)` only, and the count rules above still apply — through `.Once()` on the `On` call.
- **Synchronize before asserting on a call made off the test goroutine.** Wait for the handler to return (a done channel, a `WaitGroup`) before verifying expectations; asserting while the goroutine is still running passes or fails by scheduling, and `-count=5` will find that out.
- **testify matches expectations in declaration order.** A catch-all (`mock.Anything`) declared before a specific `mock.MatchedBy` absorbs the call and leaves the specific one unreachable. Declare the specific matcher first — and for an exactly-once assertion, do not declare a catch-all at all, since the specific expectation retires after its first call and the next one falls through. The failure reads as an **under-call in your production code** ("1 out of 2 expectation(s) were met … needs to make 1 more call(s)"), which sends you to debug the wrong side; "has been called over N times" is the different case where every matching expectation is already exhausted.

## Graceful Shutdown
- `http.Server.Shutdown` treats the two kinds of long-lived handler oppositely, and the dangerous case is the quiet one. A **hijacked** connection (WebSocket) is untracked the moment it is hijacked: `Shutdown` "does not attempt to close nor wait for" it (`net/http` documents this and points at `Server.RegisterOnShutdown` — which launches its hooks in their own goroutines and does not wait for them, so it is a notification, not a drain), so it can return immediately while every device handler is still running — the drain is not defence in depth, it is the only thing between SIGTERM and a fleet of abnormal closures. A **streaming** handler that stays active (SSE, long-poll) is the reverse: `Shutdown` waits for it, so one that blocks for the connection's lifetime makes `Shutdown` wait out its full timeout and then return an error. Both need the drain; only one will tell you it was missing. `httptest.Server.Close` untracks hijacked connections the same way, so a test's own teardown will not wait for them either.
- A service holding long-lived connections provides an explicit drain hook and invokes it **before** `Shutdown`. The drain, in order: refuse new connections, then send each tracked connection the protocol's close message, then wait a bounded grace period, then close the sockets of whatever has not gone. Refusing and registering share one critical section — a connection that passed the check but has not yet registered is missed by the sweep, which is the failure the refusal exists to prevent. The peer's acknowledgement usually arrives at the handler's own read loop rather than at the drain, so waiting for the handler goroutine to return *is* the ack wait; budgeting them separately doubles the worst case. The HTTP `Shutdown` that follows is a separate phase and gets its own budget: a drain that spends a shared one leaves `Shutdown` nothing with which to wait out a genuinely in-flight API request. (It will not *report* that as an error on a quiescent server — `Shutdown` checks for idle connections before it checks the context, so an expired context still returns nil once everything is closed — it simply gives up the wait.)
- **The drain writes to connections that are already being written to.** Send the close through the library's control-frame API, which is typically the only write documented as safe alongside other writes (gorilla: "Close and WriteControl … can be called concurrently with all other methods"); any other write path needs that connection's write mutex. This is the one place the concurrency rules above and the shutdown ordering meet, and it is the least-exercised path in the service.
- **Closing the socket is not a clean close.** Dropping the connection is what the peer sees as an abnormal closure; the close frame and the grace period are what make it graceful. A drain hook that only closes connections produces the outcome it was written to prevent.
- **Derive the shutdown context from `context.Background()`, not from the signal context.** `signal.NotifyContext`'s context is already cancelled when SIGTERM arrives, so passing it to the drain cancels every close message instantly — correct ordering, dropped connections anyway.
- State what a shutdown path actually does at its call site, including what it does *not* cover. A drain that covers only some of the work is worse documented as a drain than not documented at all.

## Module & Project Setup
- **Module path**
  If the Go module path is unknown, stop and ask:
  "What should the Go module path be? (e.g., github.com/yourname/project-name)"
  Record the confirmed go path in `plan.md`.

- **go.mod handling**
  - Check if `go.mod` already exists in the project root.
  - If `go.mod` exists do not run `go mod init`; use the existing module.
  - If `go.mod` does not exist run `go mod init [path]` using the go path specified in the plan.
  - Never create nested Go modules (only one `go.mod` at project root).

- **Module name validation**
  Perform only basic sanity checks (non-empty, no illegal characters).
  User is responsible for semantic correctness of the go path.

- **vendor/ directory**
  Never edit files inside `vendor/` directly. The directory is fully managed by `go mod vendor`, which overwrites it entirely on every run. Any manual changes are invisible to the build server and will cause build failures.

## Tooling & Build Behavior
- Always run `gofmt` (or `go fmt`) on generated code.
- Use `go mod tidy` after adding or removing dependencies.
- Test using `go test` (`go test ./...` or a more targeted path for specific changes).
- Run `go vet` after all changes.
- **Verification after Edits:** Always run `go vet` (or the project's equivalent build/verification step) after any non-trivial `replace_string_in_file` operation. This ensures that syntax errors introduced by automated edits (e.g., shell interpolation issues) are caught immediately before further implementation or testing.
- If available run `golangci-lint` before considering changes complete.
- Always run `go mod tidy` and `go mod vendor` before `go generate`.
- Always use `go generate` to generate code, never use `generate.sh` or similar.
- A `go:generate` directive runs with the working directory of the file that carries it. A generator that inspects the module (walking sources, reading `go.mod`) must locate the module root itself — `go env GOMOD` — rather than assuming the current directory is it.
- **errcheck and writes, which depends on your exclusions.** With no `exclusions:` block, errcheck's built-in list silences `fmt.Fprint*` to **`os.Stderr` only** — `os.Stdout` is flagged, as is a raw `os.Stdout.Write` — and it flags every `fmt.Fprintln`/`Fprintf` to an `io.Writer` value — so a testable CLI whose core takes writer parameters is flagged throughout. With the `std-error-handling` preset below, the whole `fmt.Fprint*` family is silenced for any writer, and what remains flagged is raw `w.Write`/`io.WriteString` on an `io.Writer` value. Decide which regime you are in before writing discard helpers: under the preset, a `func fprintln(w io.Writer, ...)` wrapper suppresses a finding you no longer get, while the calls that do need it are the raw ones.
- **golangci-lint v2 config.** The toolchain uses v2, which rejects the legacy unversioned config. Start from `version: "2"` with a `linters:` block and a separate `formatters:` block — `gofmt` and `goimports` are formatters in v2, not linters, so `golangci-lint run` *reports* formatting and `golangci-lint fmt` is what fixes it. v2 also drops the default exclusions, so without an `exclusions:` block every `defer f.Close()` in the tree is an errcheck finding on day one:

```yaml
version: "2"
linters:
  default: none
  enable:
    - bodyclose
    - errcheck
    - errorlint
    - govet
    - ineffassign
    - noctx
    - staticcheck
    - unused
  exclusions:
    generated: lax
    presets:
      - std-error-handling
formatters:
  enable:
    - gofmt
    - goimports
```

- Findings to pre-empt, most often in tests: `bodyclose` (close `resp.Body` on `net/http` client calls), `noctx` (use the `*WithContext` constructors, e.g. `httptest.NewRequestWithContext`), `errorlint` (compare sentinels with `errors.Is`, not `==`, and reach a typed error with `errors.As`). Fix and re-run one at a time: findings are reported one per line by default, so a second one can be hiding behind the first.
- **A WebSocket dial's response needs a nil guard, whatever you decide about closing it.** The dial returns a nil response on a transport failure and a non-nil one **alongside** its error on a handshake rejection, so `defer resp.Body.Close()` panics on the first and a guard placed after an early return misses the second — bind the guarded close above the `if err != nil`. Whether the close is needed at all is the library's contract to state (the mainstream ones hand back a no-op closer over memory), and whether a linter flags it varies with the call shape, so check both rather than reasoning from either.
- **The repo's `.golangci.yaml` may be weaker than CI's.** A linter absent from the local `enable:` list cannot be reproduced locally at all, so a CI-only finding is closed by matching the repo's existing annotation style, not by a local run.

## Coding Conventions (Defaults)
- Package names: lowercase, single word, no underscores.
- Exported identifiers: UpperCamelCase.
- Error handling: Use `errors.Is`/`errors.As`; wrap with `fmt.Errorf("%w", err)` when adding context.
- Testing: Prefer table-driven tests for logic with multiple cases.
- Dependencies: Minimize 3rd party imports; prefer writing standard library code when reasonable.
- JSON slice initialization: When a function returns a slice that will be marshalled to JSON, initialize it with `make([]T, 0)` rather than `var s []T`. An uninitialized slice marshals to JSON `null`; `make([]T, 0)` marshals to `[]`, which is the expected form for JSON arrays in MCP tool responses and most API contracts.
- SQL embedded in Go code: follow `skills/sql.md`. Add it to the plan's Applicable Guidelines alongside this file whenever the work touches queries, schema, or migrations.
- Comments: see `skills/comments.md` for what to cut and what earns its place. Go-specific: a comment stating a fact about *other* code's behavior ("the caller never passes nil", "X is already replicated by the time this runs") is a liability — it goes stale silently. Prefer deriving the fact locally, testing it, or asserting it so a violation fails loudly; reserve prose for constraints the code cannot express.

## Security & Safety Invariants
- Never use the `unsafe` package unless explicitly required in a feature plan.
- Validate all untrusted input (use `net/http`, middleware, or approved libraries).
- Default to `math/rand` for general-purpose randomness. Use `crypto/rand` only when the feature plan explicitly requires cryptographic randomness (e.g., token generation, secret keys). Security-sensitive usage of either package is reviewed by security specialists.

## Explicit AI Freedom
The AI has full discretion over:
- Internal variable/function naming (except user-visible APIs)
- Exact file organization and package splitting (within idiomatic Go)
- Minor refactoring for readability or performance (unless constrained by non-functional requirements)
- Test structure details (table-driven vs simple) unless the plan specifies specific coverage

## Usage
Reference this file in plan document when Go is the target language.
Follow these rules automatically unless a plan explicitly overrides them.
