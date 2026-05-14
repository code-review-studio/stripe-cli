---
title: Cobra Command Registration
model: claude-opus-4-6
reasoning: high
effort: high
input: full_diff
include:
  - "pkg/cmd/**"
exclude:
  - "pkg/cmd/**/*_test.go"
  - "pkg/cmd/resources_cmds.go"
  - "vendor/**"
conclusion: neutral
---

Review changes under `pkg/cmd/` for new Cobra commands that aren't wired up. A command that compiles but isn't registered in `root.go`'s `init` function (or its parent command's equivalent) is silently invisible — invoking it returns "unknown command". This class of bug ships through unit tests because the command's own tests construct it directly.

## What to flag

### New Commands Must Register In root.go (🔴 Must fix)

- PRs that add a new `cobra.Command` definition under `pkg/cmd/` (or a new subcommand-collecting file like the `pkg/cmd/plugin.go` → `pkg/cmd/plugins/*.go` pattern) without also adding a corresponding `rootCmd.AddCommand(...)` line in `pkg/cmd/root.go`'s `init` function (or a `parentCmd.AddCommand(...)` for subcommands). Without registration, the CLI returns "unknown command" when the new command is invoked, but unit tests that construct the command directly will still pass.

**Cite as:** New Commands Must Register In root.go
**Source:** [`ARCHITECTURE.md` § Commands](https://github.com/stripe/stripe-cli/blob/master/ARCHITECTURE.md#commands)
> Commands are registered in `pkg/cmd/root.go` in the `init` function (look for the big `rootCmd.AddCommand(...)` block). If a command isn't registered there, it'll give an error when invoked from the CLI (e.g. `stripe missing_command` gives an "unknown command" error).

## What to ignore

- Do not duplicate the built-in Correctness check — it already covers runtime bugs and logic errors.
- Do not flag issues already caught by the repository's static linters (`.golangci.yml`).
- Ignore generated code (`resources_cmds.go`), vendored dependencies, and lockfiles.
- Subcommands of a parent that *is* registered are fine — only flag commands whose parent is `rootCmd` and which aren't listed in `root.go`'s init.
- Test-only `cobra.Command` definitions inside test files are out of scope.

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
