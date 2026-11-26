---
verdict: BLOCKING
prior:
  - id: F20
    status: resolved
  - id: F21
    status: resolved
findings:
  - id: F22
    severity: BLOCKING
    class: design
    location: "_lym/discussion/decision-record.md:314"
    summary: "D13's installer clear is a function only tests call, placed in a production package (internal/fabriccli), which breaches PATTERN-test-only-code; D12 also bars the test files of fabricengine, fabriccore and fabriclayout from importing fabriccli, so a fabric test of its own unset-seam refusal cannot call it."
    fix: "In D13, drop the sentence 'The installer package also offers a clear, which unsets the required seams and returns a restore that puts back what was installed' and rewrite the two sentences after it. State that the unset-seam refusal is a fabriccore rule over a snapshot fact saying whether the seam is supplied, covered by a tier0 row. If an engine-level check is also wanted, state that each required seam's own setter in fabricengine (the one the installer calls, so reachable from main) accepts nil and returns a restore, and that a fabricengine test unsets through it, builds its hub first and does not call t.Parallel. Apply that same sequential rule to any test that sets a process-global seam. In D5, replace 'or clears the seams through the installer's clear' to match, and make the acceptance bullet 'A test shows a verb run with a required seam unset refuses before acting' name that core row."
  - id: F23
    severity: BLOCKING
    class: design
    location: "_lym/discussion/decision-record.md:309"
    summary: "The installer is told to install a required 'stencil ordering judge', but no fabric verb consumes one: fabric's StencilSource only builds a stencilstore.Source for stencilcli and cmd/lym, and fabric's second stencilstore use, StencilBaseByStamp (RelPath and BodyHash in stencilhistory.go), is covered by no seam, so the stencilstore cut and its tier0 closure guard cannot be met as designed."
    fix: "In D13, replace the internal/stencilstore bullet: StencilSource (warpforward.go) and StencilBaseByStamp (stencilhistory.go) both leave fabric for a package above fabric that internal/stencilcli and cmd/lym import. That package must not be internal/stencilstore, since leaf packages import stencilstore. It reads git through fabric's general reads only (HeadContains, plus path-taking reads for commit-exists, is-ancestor, a path's revisions and a file at a revision). Their tests (warpforward_integration_test.go's StencilSource test and stencilhistory_integration_test.go) move with them under D12. Then remove the stencil ordering judge from the required-seam bullet, the installer sentence, the 'rests on shed's codec or the stencil ordering judge' and 'calls the codec or the judge' sentences, the 'real stencil store' adapter criterion and any other mention, and add the two moved functions to the part of D9 that cuts them."
---

# Round 3 review: fabric-git-free-core discussion

F20 and F21 are resolved.
The installer gives `internal/hubforge` binaries the required seams with no new hubforge import, and F21's write sequence matches `internal/state`.
The fix text raises two new blocking defects.

### [BLOCKING] The installer's clear is test-only production code that fabric's tests cannot import

**Location:** `_lym/discussion/decision-record.md:314`

**Issue:** `PATTERN-test-only-code` says "A function only tests call lives in a `_test.go` file or a test-support package, never in a production file".
Nothing reachable from a `main` package unsets the required seams, and the record has `internal/hubforge` call the install only.
`cmd/lym/testonlycode_linux_test.go` would therefore report the clear.
No kit can hold it either, since a kit imports no `internal/*cli` package.
D12 names `internal/fabriccli` as above fabric, so the refusal, which is fabric's own behaviour, cannot be tested from fabric's packages through this clear.
D1 already puts every refusal in the core, where a made-up snapshot expresses "seam unset" with no clear at all.

### [BLOCKING] The required stencil ordering judge has no fabric consumer, and the second stencilstore use has no home

**Location:** `_lym/discussion/decision-record.md:309`

**Issue:** `internal/fabricengine` imports `internal/stencilstore` in `warpforward.go` and `stencilhistory.go` only.
`StencilSource` has no caller inside fabric: its callers are `cmd/lym/stencilseed.go` and `internal/stencilcli/cli.go`.
It returns a `stencilstore.Source`, so an injected judge cannot keep it in fabric without the import.
No fabric verb "would use" the judge, so the required-seam refusal, the installer entry and the "flow whose outcome rests on ... the stencil ordering judge" seam tests describe nothing.
`StencilBaseByStamp`, called by `internal/stencilcli/diff.go`, uses `stencilstore.RelPath` and `stencilstore.BodyHash`, which an ordering verdict cannot supply.
`PATTERN-fabric-boundary`'s isolation section also says a seam only one module could use belongs in that module, "which drives fabric through its general verbs".

## Focus results

- Required seams in test binaries: settled by the installer for every binary that builds a hub through `internal/hubforge`, with hubforge's import set unchanged.
  Reachability fails for the clear only (F22).
- Nil workspace builder: `internal/ideengine` and `internal/beamcli` tests that expect the file fail loudly without the builder.
  No acceptance criterion catches a vacuous pass; the cases are listed below.
- Shed records Writes: holds.
  `state.WriteJSON` locks, reads, keeps undeclared keys and writes under one lock, the takeover write stays a one-key patch, and a criterion checks the bytes with an undeclared key present.
- Open risks against the added entries: they do not agree (first note below).

## Notes below this round's bar

These sit on unchanged lines or are under BLOCKING, so this incremental round files none of them as findings.

- `## Open risks` in `decision-record.md` still carries all three lines that both added arbiter entries tell the writer to drop.
  `support-log.md` still marks Q12 and Q15 "open, for the parent".
  The rubric calls this a `consistency` defect, so the parent or the fixer should still apply it.
- `internal/machinecli/start_test.go` only computes the workspace path to compare strings, over made-up hubs, and reads no file.
  D13's list of reading tests is wrong about it.
- Without the builder, some `internal/ideengine` negative assertions pass vacuously:
  the "workspace file unchanged by a task seed and spawn" check and the "no beam file for a beam with no readable origin record" check in `spawn_hub_integration_test.go`.
- `internal/beamcli/dobbyleftover_integration_test.go` calls `t.Parallel`, and the builder it must install is process-global.
- D1 closes `internal/fabriccore` to the standard library and `internal/fabriclayout`, while D2 and D12 put the secret-path refusal in the core.
  The matcher is `internal/secretpath`, so the record needs either that import or a snapshot fact carrying the engine's matches.
- D13 says a seam reaches fabric "the way `fabricengine.OriginSeams` does today", which is a per-call struct, and also that each seam is installed in the process at start.
  A fabric test that sets a process-global path seam to its own hub would affect parallel tests.

## Rejected

- Fabric writing the `takeover` key with its own entry type as a breach of "fabric learns nothing of ... shed": the entry is fabric's own vocabulary under D13, the path comes from the seam, and no shed package is imported.
- A re-executed test subprocess meeting the required-seam refusal: the one in `internal/fabriccli/cleanhub_integration_test.go` runs only `pairs`.
