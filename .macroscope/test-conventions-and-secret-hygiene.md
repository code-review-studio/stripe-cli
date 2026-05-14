---
title: Test Conventions & Secret Hygiene
model: claude-opus-4-6
reasoning: high
effort: high
input: full_diff
include:
  - "**/*_test.go"
exclude:
  - "vendor/**"
  - "**/*.pb.go"
  - "**/*_generated.go"
conclusion: neutral
---

Review changed test code for project test-conventions violations and (in `canary/`) secret-leakage risks. Flag tests that bypass the project's sanitized-logging helpers, use stdlib mocking when established libraries are available, or skip the canary prefix/isolation conventions.

## What to flag

### Sanitized Logging In Canary Tests (🔴 Must fix)

- In `canary/**/*_test.go`, any use of `t.Fatalf`, `t.Errorf`, or `t.Logf` instead of the project's sanitized helpers `fatalf`, `errorf`, `logSanitizedf`. The sanitized wrappers automatically redact `sk_test_*`, `sk_live_*`, `rk_*`, `whsec_*`, and bearer tokens before logging — bypassing them risks publishing secrets to public GitHub Actions logs.

**Cite as:** Sanitized Logging In Canary Tests
**Source:** [`canary/README.md` § Adding New Tests](https://github.com/stripe/stripe-cli/blob/master/canary/README.md#adding-new-tests)
> **Always use sanitized logging**: Use `fatalf()`, `errorf()`, and `logSanitizedf()` instead of `t.Fatalf()`, `t.Errorf()`, and `t.Logf()`

### Use testify/require Or testify/assert (🟡 Should fix)

- New tests that use stdlib `testing.T` patterns (e.g., `if got != want { t.Errorf(...) }`) where the project's convention is `testify/assert` and `testify/require`. The CLI codebase is consistent on this.

**Cite as:** Use testify/require Or testify/assert
**Source:** [`CLAUDE.md` § Key Conventions](https://github.com/stripe/stripe-cli/blob/master/CLAUDE.md#key-conventions)
> **Testing**: Use `testify/assert` and `testify/require`. Mock HTTP with `httptest.NewServer`. Mock filesystem with `spf13/afero`

### Mock HTTP With httptest.NewServer (🟡 Should fix)

- New tests that stand up custom HTTP mocks (raw `net.Listen`, third-party mock libraries, hand-rolled handlers) when `httptest.NewServer` would work. The codebase is consistent on this for Stripe API mocking.

**Cite as:** Mock HTTP With httptest.NewServer
**Source:** [`CLAUDE.md` § Key Conventions](https://github.com/stripe/stripe-cli/blob/master/CLAUDE.md#key-conventions)
> Mock HTTP with `httptest.NewServer`.

### Mock Filesystem With spf13/afero (🟡 Should fix)

- New tests that touch the real filesystem (`os.WriteFile`, `os.MkdirAll`, `ioutil.TempDir`, `os.Create`) for unit tests where `spf13/afero` (already used throughout the codebase) would isolate the test.

**Cite as:** Mock Filesystem With spf13/afero
**Source:** [`CLAUDE.md` § Key Conventions](https://github.com/stripe/stripe-cli/blob/master/CLAUDE.md#key-conventions)
> Mock filesystem with `spf13/afero`

### Canary Test Naming Conventions (🟡 Should fix)

- In `canary/`, new test functions that don't follow the prefix conventions: `TestOffline*` for tests that work without an API key, `TestAPI*` for tests requiring `requireAPIKey(t)`. Mixing or omitting these prefixes confuses CI about which tests need secrets.

**Cite as:** Canary Test Naming Conventions
**Source:** [`canary/README.md` § Adding New Tests](https://github.com/stripe/stripe-cli/blob/master/canary/README.md#adding-new-tests)
> Use `TestOffline` prefix for tests without API requirements

### Isolated Config Dirs In Canary Tests (🟡 Should fix)

- In `canary/`, tests that touch the user's real `~/.config/stripe/` (e.g., via the default config path) instead of an isolated temp dir via `testutil.CreateTempConfigDir()`. Without isolation, tests step on each other and on developer machines.

**Cite as:** Isolated Config Dirs In Canary Tests
**Source:** [`canary/README.md` § Adding New Tests](https://github.com/stripe/stripe-cli/blob/master/canary/README.md#adding-new-tests)
> Use isolated config directories via `testutil.CreateTempConfigDir()`

## What to ignore

- Do not duplicate the built-in Correctness check — it already covers runtime bugs and logic errors.
- Do not flag issues already caught by the repository's static linters (`.golangci.yml` — golangci-lint v2 with staticcheck, govet, unused, ineffassign, misspell, dupl).
- Ignore generated code, vendored dependencies, and lockfiles.
- Pre-existing tests that don't follow these conventions are out of scope — only flag what the diff adds or modifies.

## Output format

Group findings by severity (🔴 Must fix, 🟡 Should fix, 🟢 Nit). For each finding, post an inline review comment on the offending line.

Every inline comment must end with a collapsible citation block pointing back to the original rule in this repository's own docs. Identify which `###` rule above the finding violates, then build the footer from that rule's `Cite as` / `Source` / quote — verbatim, no paraphrasing. Insert a blank line between the comment body and the `<details>` block.

```
<details><summary><em>Violates</em>: {Cite as value}</summary>

**Source:** [{path} § {section}]({anchored_url})

> {verbatim quote}

</details>
```

After inline comments, post a top-level PR comment with a one-line summary per finding. If no issues are found in the changed code, post a single top-level comment: **"All clear."**

If no issues are found, report **"All clear."** Do not invent issues to fill space.
