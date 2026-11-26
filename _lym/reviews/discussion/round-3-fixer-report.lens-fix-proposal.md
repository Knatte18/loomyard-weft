```yaml
lens: fix-proposal
outcome: reported
```

Task: test the reviewer's proposals for F22 and F23 that the fixer carried out, and look for cases where they are wrong or break something.

**MEDIUM: no test reaches the engine half of the unset-seam refusal (F22's proposal)**
- Changed text: D13's new sentence "The refusal is a rule in `internal/fabriccore` (new) over a snapshot fact saying whether each required seam is supplied, and a tier0 core row covers it, so no test unsets an installed seam." Also the rewritten acceptance bullet "A tier0 core row shows a verb whose snapshot marks a required seam unsupplied refuses in its pre-lock phase…".
- The failing case: an engine reader that always sets the "supplied" fact to true, even with the seam nil, passes every criterion. The core row uses a made-up snapshot. The `cmd/lym` criterion and the hubforge remove criterion run with seams installed. D5 now says only "a test that builds no hub meets the refusal in the engine", and no criterion asks for that test. So nothing checks that the engine turns a nil seam into "unsupplied", or that the verb prints the refusal naming the seam before acting.
- Evidence: the review's own fix offered an engine-level check through the nil-accepting setter ("If an engine-level check is also wanted…"), and the fix took the core-only branch. A tier0 or tier1 engine test of a verb on a hub-free location with the seam nil would close this, and D5's sentence already allows such a test.

**LOW: the sequential rule now catches tests that only meant to supply their own path seam (F22's proposal)**
- Changed text: "A test that sets a process-global seam, the workspace builder included, does so through the seam's own setter … and does not call `t.Parallel`".
- The case: D13's unchanged line "A fabric test that sets a seam itself supplies a path or lock-path seam with a location in its own hub" now forces every such fabric test to run sequentially. That moves fabric tier1 time to serial, against D10's target. Nothing tells the plan writer to prefer hubforge's installed seams over setting one.
- Evidence: the record's D10 target is "a cut of at least half in both fabric packages' tier1 time". The rule applies to any fabric test that sets a seam.

**LOW: the moved `StencilBaseByStamp` needs fabric names that are unexported today (F23's proposal)**
- Changed text: the new `internal/stencilstore` bullet in D13, "It reads git through fabric's general reads only…".
- The case: `StencilBaseByStamp` builds its path from `BoltDir(hub)` and the unexported `stencilsDirName` (`internal/fabricengine/stencilhistory.go:29-30`). `internal/stencilsource` cannot reach `stencilsDirName`. The bullet names the git reads fabric adds but not the path names it must export, through `internal/fabriclayout` (new) or the existing `StencilsDir`/`StencilsSubtreeRel`.
- Evidence: `internal/fabricengine/stencilhistory.go:29`, and `cmd/lym/stencilseed.go:119` already uses `fabricengine.StencilsDir(hub)`.

I made no further tool calls.
