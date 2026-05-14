---
title: Generated Code Guardrails
model: claude-opus-4-6
reasoning: high
effort: high
input: full_diff
include:
  - "pkg/cmd/**"
  - "pkg/proxy/**"
  - "pkg/requests/**"
  - "pkg/gen/**"
exclude:
  - "**/*_test.go"
  - "vendor/**"
conclusion: neutral
---

Review changes that touch auto-generated code. Flag direct edits to generated files (which will be overwritten on the next `go generate` run), missing generated-file headers, and PRs that try to modify the generated `events` command in-place instead of using the manual override pattern.

## What to flag

### No Edits To resources_cmds.go (🔴 Must fix)

- Any diff that modifies `pkg/cmd/resources_cmds.go` directly. This file is regenerated from `api/openapi-spec/spec3.cli.json` via `go generate ./...` — manual edits will be overwritten on the next code-gen run. To change a command's behavior, add a manual override file under `pkg/cmd/resource/` (the pattern `pkg/cmd/resource/events.go` already uses).

**Cite as:** No Edits To resources_cmds.go
**Source:** [`CLAUDE.md` § Project Structure](https://github.com/stripe/stripe-cli/blob/master/CLAUDE.md#project-structure)
> `pkg/cmd/resources_cmds.go` - **Auto-generated** from OpenAPI spec (do not edit manually)

### No Edits To events_list.go (🔴 Must fix)

- Any diff that modifies `pkg/proxy/events_list.go` directly. It is regenerated from the OpenAPI spec via `go generate ./...` and edits will be overwritten.

**Cite as:** No Edits To events_list.go
**Source:** [`CLAUDE.md` § Project Structure](https://github.com/stripe/stripe-cli/blob/master/CLAUDE.md#project-structure)
> `pkg/proxy/events_list.go` - **Auto-generated** event type list

### Override Generated Commands, Don't Edit Them (🔴 Must fix)

- PRs that change a generated resource command's behavior by editing the generated definition in `pkg/cmd/resources_cmds.go`. The pattern is to create a manual file under `pkg/cmd/resource/` whose name shadows the generated command (e.g., `pkg/cmd/resource/events.go` takes precedence over the generated `events` command).

**Cite as:** Override Generated Commands, Don't Edit Them
**Source:** [`ARCHITECTURE.md` § Auto-Generated Resources](https://github.com/stripe/stripe-cli/blob/master/ARCHITECTURE.md#auto-generated-resources)
> Generated commands can be manually overridden, as we do with `pkg/cmd/resource/events.go` taking precedence over the generated `events` command.

### Generated Files Need DO NOT EDIT Header (🟡 Should fix)

- New files added under generator output paths (or new files matching the auto-gen pattern, e.g., output of a new `//go:generate` directive) that lack a `// This file is generated; DO NOT EDIT.` header at the top. Without it, the next contributor can't tell the file is regenerated.

**Cite as:** Generated Files Need DO NOT EDIT Header
**Source:** [`CLAUDE.md` § Key Conventions](https://github.com/stripe/stripe-cli/blob/master/CLAUDE.md#key-conventions)
> generated files have `// This file is generated; DO NOT EDIT.` header

## What to ignore

- Do not duplicate the built-in Correctness check — it already covers runtime bugs and logic errors.
- Do not flag issues already caught by the repository's static linters (`.golangci.yml` — golangci-lint v2 with staticcheck, govet, unused, ineffassign, misspell, dupl).
- Ignore vendored dependencies and lockfiles.
- Edits to the *templates* in `pkg/gen/*.go.tpl` are in scope (they drive the generators); edits to the generated output files (`resources_cmds.go`, `events_list.go`, `stripe_version_header.go`) are what these rules flag.
- Test files are out of scope for this agent.

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
