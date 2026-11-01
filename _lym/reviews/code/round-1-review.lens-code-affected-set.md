```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set — what the diff forces outside its touched lines.

**Per-change set**

1. Wave text in `burler-fork-reading.md` (lines 19-26).
   - Stencil consumers: `contracts/stencils/stencils.go:194,552` and `markers_test.go:29` (declares no markers; the added text has none), plus `prompt.go:394-401` `forkReadingBlock` (fills no new marker). Covered.
   - Tests: `template_test.go:76,184` and `prompt_test.go:410` assert only the heading, placement and marker absence, so none breaks. Covered.
   - Sibling stencils: the explore step (`burler-step-1-explore.md:39`) carries "Make the reads your focus needs … in one message". The fan block is injected after it and now says the same thing with more detail, with no contradiction. The fixer template and `burler-fix-self-check.md` are unrelated. Covered.
   - Pre-existing rules (`burler-fork-reading.md:28-34`): "Read definitions and call sites, not whole files", "Judge nothing while you read" and the manifest-close rule are all intact. Covered.

2. `doc.go:286`.
   - It restates the stencil's behavior in one sentence inside the cluster-round paragraph, and it matches the stencil. Covered.
   - It uses "a few waves", which is a loose tally. The stencil itself says "waves" without a count. LOW/NIT at most, and I raise no finding.

3. PATTERN fit.
   - `PATTERN-stencil-ownership` is satisfied: the prose lives in the stencil, read at call time, with no Go composition.
   - The commit declares no `Pattern-Departure:` trailer, and none is needed.
   - The directive declares no PATTERN changes, so nothing is owed. Covered.

**Findings**

- F-A, LOW, consistency, `burler-fork-reading.md:19-26`. The checklist has no wave-1 item for the third bullet, the "callers and callees of each changed symbol", or the fourth bullet's tests and docs. Wave 1 covers them with "one search per symbol for its callers, tests, docs and fixtures", but it names no callees, and no wave names "every path that reaches it from another process" or "the failure, restart and resume paths". Wave 3 mentions only "a spawned process or a restart path", and only as a hop the second wave opened. Fix: add "callees" to the per-symbol search in wave 1, and let wave 3 name failure and resume paths. Evidence: the checklist at lines 11-17 against wave 1 at line 22.

- F-B, LOW, consistency, `burler-fork-reading.md:22` against `burler-step-1-explore.md:39`. The wave-1 phrase "the read of each file the target names" conflicts mildly with "Read definitions and call sites, not whole files" at line 28, because it can read as whole-file reads. Checklist item 1 already says "at the definitions of the symbols it cites", and the wave text drops that qualifier. Fix: write "the read of each file the target names, at the definitions of the symbols it cites".

I found no missing doc, test or PATTERN update elsewhere in the set.

**Tool calls**
1. grep over the burler stencils, the burlerengine tests and `ls pattern/PATTERN-stencil-ownership.md`: it confirmed the other batching sentences and which tests touch `fork_reading`. My context lacked the stencil-wide grep and the test list.
