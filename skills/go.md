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

### Running tests
- *NEVER* use `-short` with `go test`. There is no plan-level override for this.
- *NEVER* run `go build` to test; use `go test` (or `go run` if you want to interact with a running instance). The one check that needs a built binary is a process's exit code: `go run` exits 1 whatever non-zero code the program returns, so build it (`go build -o <gitignored dir>/<name>`) and run the binary to verify exit codes. Source: WI 1249.
- Run the project's documented gate before making changes, to know the state you start from, and again after all changes for final verification — `go test ./...` where the project documents none. A plain `go test ./...` on a project whose gate runs `-race` exits 0 over a race that gate fails (*Run the documented gate*, below). Source: WI 1713.
- **Send a gate's output to a file and read the file** — `go test ./... > <gitignored dir>/gate.log 2>&1; echo "exit=$?"` — never pipe the run itself into `tail`, `head` or `grep`; filter the saved file afterwards. A pipeline's exit status is the filter's, so a failing run reports success; and a filter that truncates a race or panic report drops its head, the part that names the cause. Where a pipe is unavoidable, report `${PIPESTATUS[0]}`. Source: WIs 1618, 1705.
- Pass `-timeout` on any run that can block: the default is generous enough that a hung package looks like a slow one, and the timeout is what turns it into a goroutine dump. `-timeout` applies to each package's test binary, not to the run, and `-count=N` runs N times inside that binary. So a whole-module gate takes a `-timeout` of about twice its slowest package's usual run time under the gate's own flags (`-race` included), times N, unless the project documents a value.
- **Run the package of every file the change touched before the whole-module gate.** After the targeted runs, `go test ./path/to/pkg/...` for each touched package: the tests written for the change pass while an older test in the same package, built on the behaviour the change replaced, fails — and the whole-module gate is the slow place to find that out. Source: WIs 1560, 1288.
- **The first whole-package run after a change to a state machine or a protocol takes a `-timeout` of two to three times the package's usual run time** — `-timeout 150s` for a package that takes 60 s. A hang is the likeliest new failure of such a change, and it is as likely in an older test as in the new ones; under the default, or a gate's whole-module timeout, it costs ten minutes or more before the goroutine dump appears. Source: WI 1346.
- **Run a whole-module gate from a detached worktree of the commit under test** (`git worktree add --detach <dir> <commit>`, with `<dir>` under a gitignored directory), whenever the tree will be edited while the gate runs. That is certain for a gate started before Code Review, Test and Voice are done, because each of them edits the tree by design: start such a gate from a worktree of a work-in-progress commit, or hold it until Voice is done. Commit the change first — a snapshot such as `git stash create` omits untracked files, new tests included. The result certifies that commit and is recorded with its hash; a later commit that touches anything the build or the tests read needs its own gate. An edit to such a file in the tree a gate is running from makes the result describe no tree that ever existed, and the run is discarded. Remove the worktree (`git worktree remove`, `--force` when the run left files) once the result has been read and a failure diagnosed. Source: WIs 1653, 1657, 1705.
- `-race -count=5` on any new or changed concurrent package, before it is called stable; why is in `skills/go/testing.md`.
- **Run every `-race` gate with `GORACE=log_path=<absolute path>`**, which changes where reports go and nothing the gate tests. Each test binary then writes its race reports to `<path>.<pid>`, and the test output carries only `race detected during execution of test` — read the files. A relative path lands in each package's own directory. An intermittent race may reproduce once, and its report is the whole diagnosis. Source: WI 1705.
- **Run the documented gate, not a faster decomposition of it.** If the project documents one whole-module command as its gate, run that command. Splitting it into parallel halves changes what is tested — inter-package contention is part of what the combined run exercises, and a split has hidden a real regression for a whole session. If the combined command is too slow, record that fact; it is not a licence to substitute the halves.

### Writing tests
- `TestMain` must live in a `_test.go` file. A `TestMain` in a regular file compiles without complaint, passes `go vet`, and never runs — no gate in this document catches it.
- **A wait on a counter asserts `>=`, never `==`.** Anything else that can increment the counter — a retry, a background tick, a second node — makes `==` a wait that steps over its own target and times out. Where the exact count is the behaviour under test, assert it once, after the wait, and after the test has stopped every other incrementer or driven it to completion (an injected tick, a stopped service) — an exact check straight after the wait races the next increment. Source: WIs 1346, 1627.
- **A call that drains what it returns sits in the loop body, never in the loop condition.** A `Take*`, a `Drain`, a channel receive: each clears the state it reports, so `for len(sub.TakeDeliveries()) < 2 {` consumes the deliveries it is counting and the test can never fail. Accumulate what each call returns, then assert on the accumulation. Source: WI 1487.
- **Package-level state a test mutates is saved and restored.** A narrowed port window, a swapped clock, a shortened timeout: restore it in `t.Cleanup`, or `-count=5` runs inherit each other's leftovers and go red only intermittently; a test that mutates it does not call `t.Parallel()`. Source: WI 1488.
- **Expressing a negative control's break in Go** — `Test.md` step 3 carries the rule; this is the form it takes here. Flip the condition (`if false && err != nil`; wrap a condition with more than one operand whole, `if false && (a || b)`, because `&&` binds tighter than `||` and a bare prefix disables only the first operand), assign the zero value (`name = ""`), discard a result (`_ = n`), or return early. Delete a block only where the deletion leaves nothing unused: a stranded variable, parameter or import fails the build, and `go test` then reports a build failure where a test result was wanted. Source: WIs 1644, 1646, 1667, 1668, 1669, 1275.
- Concurrent harnesses, benchmarks at scale, and goroutines a test blocks: `skills/go/testing.md`. Mocks: `skills/go/mocks.md`. Handlers that live for a connection, drains and shutdown: `skills/go/handlers.md`.

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
- errcheck regimes, the golangci-lint v2 configuration and the findings the gate raises: `skills/go/lint.md`.

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

## Sub-files

| Applies when the work item… | Read |
|---|---|
| writes or changes a test of concurrent code, a benchmark, or a test that blocks a production goroutine | `skills/go/testing.md` |
| uses generated or hand-built mocks | `skills/go/mocks.md` |
| touches a handler that lives for a connection's lifetime, a drain, or shutdown | `skills/go/handlers.md` |
| changes lint configuration, writes through `fmt.Fprint*` or `w.Write` to an `io.Writer` in a project that runs golangci-lint, or a gate raises errcheck, bodyclose, noctx or errorlint findings | `skills/go/lint.md` |

This table decides: a plan names the sub-files that apply in Applicable Guidelines beside this file.

## Usage
Reference this file in plan document when Go is the target language.
Follow these rules automatically unless a plan explicitly overrides them.
