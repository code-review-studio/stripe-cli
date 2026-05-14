---
title: Plugin Runtime Security
model: claude-opus-4-6
reasoning: high
effort: high
input: full_diff
include:
  - "pkg/plugins/**"
exclude:
  - "pkg/plugins/**/*_test.go"
  - "vendor/**"
conclusion: neutral
---

Review changes to the plugin Node.js runtime install/verification path. Flag drift from the hardcoded-checksum offline-verification security model: non-LTS Node.js versions, network-fetched checksums, or download paths that bypass verification.

## What to flag

### Only LTS Node.js Versions Allowed (🟡 Should fix)

- Additions to `nodeRuntimeConfigs` (or similar runtime-version map) that include non-LTS Node.js versions (odd-numbered major versions). Only LTS lines (currently 20.x; 22.x and 24.x as they enter LTS) are supported — accepting non-LTS versions risks installing Node lines that won't receive security patches over the plugin's life.

**Cite as:** Only LTS Node.js Versions Allowed
**Source:** [`pkg/plugins/RUNTIME.md` § Supported Node.js Versions](https://github.com/stripe/stripe-cli/blob/master/pkg/plugins/RUNTIME.md#supported-nodejs-versions)
> Only LTS (Long-Term Support) versions of Node.js are supported

### Hardcoded Checksums, No Network Lookup (🔴 Must fix)

- Code that fetches SHA256 checksums from `nodejs.org` (or anywhere else) at runtime — via HTTP, a config file shipped separately, an environment variable, etc. — instead of reading them from the hardcoded `nodeRuntimeConfigs` map. Hardcoding is the security model: a compromised `nodejs.org` mirror cannot subvert a checksum that was baked into the binary at build time.

**Cite as:** Hardcoded Checksums, No Network Lookup
**Source:** [`pkg/plugins/RUNTIME.md` § Hardcoded Configurations](https://github.com/stripe/stripe-cli/blob/master/pkg/plugins/RUNTIME.md#hardcoded-configurations)
> Runtime versions and checksums are hardcoded in `runtime.go`:

### Verify All Runtime Downloads Against Checksums (🔴 Must fix)

- Any new runtime-download path (new download helper, new branch in the install flow, retry logic that re-downloads, cache fill, etc.) that doesn't pass through the SHA256 verification step before extracting the archive. Every downloaded archive must be verified against the hardcoded SHA256 before being unpacked.

**Cite as:** Verify All Runtime Downloads Against Checksums
**Source:** [`pkg/plugins/RUNTIME.md` § Checksum Verification](https://github.com/stripe/stripe-cli/blob/master/pkg/plugins/RUNTIME.md#checksum-verification)
> All downloaded runtimes are verified against hardcoded SHA256 checksums from the official Node.js releases

## What to ignore

- Do not duplicate the built-in Correctness check — it already covers runtime bugs and logic errors.
- Do not flag issues already caught by the repository's static linters (`.golangci.yml`).
- Ignore vendored dependencies and lockfiles.
- Test files are out of scope for this agent.
- Plugin-binary checksum verification (the existing `Sum` field in `plugins.toml`) is out of scope for this rule set — these rules concern Node.js *runtime* download/verification specifically.

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
