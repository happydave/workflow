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

## Module & Project Setup
- **Module path**
  If the Go module path is unknown, stop and ask:
  "What should the Go module path be? (e.g., github.com/yourname/project-name)"
  Record the confirmed go path in `plan.md`.

- **go.mod handling**
  - Check if `go.mod` already exists in the project root.
  - If it exists do not run `go mod init`; use the existing module.
  - If it does not exist run `go mod init [path]` using the go path specified in the plan.
  - Never create nested Go modules (only one `go.mod` at project root).

- **Module name validation**
  Perform only basic sanity checks (non-empty, no illegal characters).
  User is responsible for semantic correctness of the go path.

## Tooling & Build Behavior
- Always run `gofmt` (or `go fmt`) on generated code.
- Use `go mod tidy` after adding or removing dependencies.
- Test using `go test` (`go test ./...` or a more targeted path for specific changes).
- Run `go vet` after all changes.
- If available run `golangci-lint` before considering changes complete.

## Coding Conventions (Defaults)
- Package names: lowercase, single word, no underscores.
- Exported identifiers: UpperCamelCase.
- Error handling: Use `errors.Is`/`errors.As`; wrap with `fmt.Errorf("%w", err)` when adding context.
- Testing: Prefer table-driven tests for logic with multiple cases.
- Dependencies: Minimize external packages; prefer writing standard library code when reasonable.

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
