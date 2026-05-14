---
title: Error Handling Conventions
model: claude-opus-4-6
reasoning: high
effort: high
input: full_diff
include:
  - "**/*.go"
exclude:
  - "**/*_test.go"
  - "vendor/**"
  - "pkg/cmd/resources_cmds.go"
  - "pkg/proxy/events_list.go"
  - "pkg/requests/stripe_version_header.go"
  - "**/*.pb.go"
  - "**/*_generated.go"
conclusion: neutral
---

Review changed non-test Go code for project-specific error-handling patterns. Flag ad-hoc errors where a typed error exists, string-comparison error checks instead of `errors.As`, and user-facing error output going to stdout instead of stderr.

## What to flag

### Use Typed Errors With errors.As (🟡 Should fix)

- New error-handling paths that construct ad-hoc errors via `errors.New` / `fmt.Errorf` and check them by string comparison or sentinel `errors.Is`, when a typed error like `RequestError` already exists in the codebase. Check typed errors with `errors.As`, not string matching.

**Cite as:** Use Typed Errors With errors.As
**Source:** [`CLAUDE.md` § Key Conventions](https://github.com/stripe/stripe-cli/blob/master/CLAUDE.md#key-conventions)
> **Error handling**: Use typed errors (e.g., `RequestError`), check with `errors.As`. User-facing errors go to stderr

### User-Facing Errors Go To stderr (🟡 Should fix)

- User-facing error output written via `fmt.Println` / `fmt.Printf` / `os.Stdout.Write` (or `cmd.OutOrStdout()`) when the destination should be `os.Stderr` (or the Cobra command's `ErrOrStderr()`). Errors on stdout break pipelines (`stripe foo | jq ...`) by interleaving error text with parseable results.

**Cite as:** User-Facing Errors Go To stderr
**Source:** [`CLAUDE.md` § Key Conventions](https://github.com/stripe/stripe-cli/blob/master/CLAUDE.md#key-conventions)
> User-facing errors go to stderr

## What to ignore

- Do not duplicate the built-in Correctness check — it already covers runtime bugs and logic errors.
- Do not flag issues already caught by the repository's static linters (`.golangci.yml` — golangci-lint v2 with staticcheck, govet, unused, ineffassign, misspell, dupl).
- Ignore generated code, vendored dependencies, and lockfiles.
- Test files are out of scope for this agent — the Test Conventions & Secret Hygiene agent handles those.
- Internal/debug logging via `log.Logger` (not user-facing) is out of scope.

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
