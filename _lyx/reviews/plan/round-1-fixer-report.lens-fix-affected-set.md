```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran each set-premise finding's search to check the fix covers every member.

Unchecked: F2, F3, F8, F10 and F12 have no set premise. F9's verb set (accept, compare, capture --side) is covered by what my context already holds, so I ran no search for it.

Checked, all covered: F1 (RunCapture callers in cards 6, 7, 9 and 14; card 5's side capture no longer inherits a job hold through "runs as above"), F4, F5, F6 (Commit callers in cards 9, 10, 11 and 14), F7 and F11.

**F4 search** (tests pinning the loom and darn edges card 14 changes):
- `internal/loomrecipe/progress_test.go:64` is covered by the fix.
- `internal/loomcli/settledguard_test.go:29` builds its own literal producer list, so it doesn't depend on the recipe and isn't affected.
- `internal/loomrecipe/escalation_test.go:128` runs darn under the kit's template. `drift.enabled` is false there, so the Drift rows are skipped and Code-Bouncer is still reached. Not affected.

**F5 search** (stale dirty-tree wording across the plan): only the rewritten sentences in `00-overview.md:63` and `05-capture-run.md:68` mention removing, and both say "never removing". No old "removing or stashing" or "cleaning the listed paths" wording is left in any card.

**F11 search** (every full `gates:` example or key list naming `code-burler-fix`): the full-key examples are `internal/gatecli/doc.go`, `docs/dotnet-hub.md`, `internal/gateslot/doc.go`, `internal/gateslot/template.yaml` and `pattern/PATTERN-test-speed.md`. Card 3 now covers all of them. `loom-status-spec.md`, `landingshed/doc.go` and the code and test files hold single-key mentions, not key lists.

**F7 search** (other Extends refs that rewrite text): the remaining Extends refs only add. These are card 1's and card 7's module-tree line in `docs/overview.md`, card 9's new bullet in `pattern/PATTERN-daemon-wakeups.md`, card 14's new section in `internal/shedrecipe/doc.go`, and card 3's new row, block line and example line. The `PATTERN-daemon-wakeups.md` list is bulleted (line 43 area), so a new entry adds and rewrites nothing.

Findings:
- **LOW.** The changed text is card 11's site sentence for `pattern/PATTERN-gate-slot.md` (moved to Edit by F7). The fix covers the PATTERN site list but leaves out its twin in `internal/loomshed/doc.go:40`. That sentence lists each row that runs its own key's `lyx gate test --wait --gate &lt;key&gt;` call ("the `code-burler-fix` key at Code-Burler-Fix and the `darn-write` key at Darn-Write, so each fix row and writer runs its own key and shares no pass with Webster's end gate"). `Drift-Verify` now runs its own `drift-verify` key through `loomshed.NewVerifyGate`, so that list becomes incomplete. No card names `internal/loomshed/doc.go`.

Tool calls:
1. grep of the tests for the edge constants card 14 changes: my context lacked the tests pinning those edges.
2. sed of `settledguard_test.go` and `escalation_test.go`: needed to judge whether each hit depends on the recipe.
3. grep for `code-burler-fix` across the repository, plus a grep of the plan for dirty-tree wording: needed to rerun the F11 and F5 set searches.
4. grep of the remaining `code-burler-fix` hits and `PATTERN-daemon-wakeups.md`: needed to classify each hit and check how the daemon-wakeups list is shaped.
