---
name: go-lint
description: Use when a Go work item changes lint configuration or a gate raises errcheck, bodyclose, noctx or errorlint findings — errcheck regimes, the golangci-lint v2 config, findings to pre-empt
---
# Go: Lint

Sub-file of `skills/go.md`; its tooling rules apply first.

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

- `golangci-lint config verify` accepts or rejects the config and is silent on success — run it after any toolchain upgrade rather than trusting that a block copied from here still parses.
- Findings to pre-empt, most often in tests: `bodyclose` (close `resp.Body` on `net/http` client calls), `noctx` (use the `*WithContext` constructors, e.g. `httptest.NewRequestWithContext`), `errorlint` (compare sentinels with `errors.Is`, not `==`, and reach a typed error with `errors.As`). Fix and re-run one at a time: findings are reported one per line by default (`issues.uniq-by-line`), so a second one can be hiding behind the first.
- **The repo's `.golangci.yaml` may be weaker than CI's.** A linter absent from the local `enable:` list cannot be reproduced locally at all, so a CI-only finding is closed by matching the repo's existing annotation style, not by a local run.
