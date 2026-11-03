```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, I looked for what reads, asserts or documents it outside the lines the diff touched.

**Per-change list**

| Change | Set found | Covered? |
|---|---|---|
| Gated, all-files-present, shell-only waiting turn end waits and nudges, then counts Done at the bound (`pollEventsTick`, `expiredTurnEnd`) | Docs: `wait.go` header, `pollEventsTick`, `expiredTurnEnd`, `ShellWaitBound`; `shuttleengine/doc.go:429-442`; `websterengine/doc.go`; `config.go:38`; `template.yaml:8`. Tests: `waiting_test.go` rows. Readers: `waitmark.go` `shellWaitActive` / `syncShellWait`; `run.go` field comments; `spec.go:124`; `websterengine/runlevel.go:39`; `claudeengine/doc.go:312`; `claudeengine/testdata/corpus/master-fork-running/case.yaml:12,43` | Partly. The diff updated the first group. The readers are covered only where noted below. |
| Gated run with a missing output file is held at once | `shuttleengine/doc.go:429`; `claudeengine/doc.go:312`; corpus `case.yaml` | Yes. This behaviour is unchanged and those texts are still accurate. |
| `BackgroundShellWaitMin` now also expires the gated Done | `config.go`, `template.yaml`, `doc.go`, `ShellWaitBound`, `websterengine/doc.go` | Yes. |
| New `shellNudge` and `nudgedShells` fields | `run.go` declares them. Nothing else reads or resets them. | Yes. |
| New `shellFirstSeen` use by the gated path | `waitmark.go` `shellWaitActive` | No. See F1. |
| Gate slot claim in the summary | `internal/gateslot/pool_integration_test.go:16` has `TestAcquire_HolderProcessDeathFreesSlot` | Yes. The test exists. |
| PATTERN entries | The change touches shuttle wait logic and agent messaging. It sends no message to another session. It spawns nothing and uses no filesystem link. `PATTERN-pane-text-injection` is the only candidate. It governs typing into an agent pane through reed's guarded paste-submit. `sendWithin` is the existing gated-send path, so it fits. | Yes. I found no violation. |
| Commit messages and departures | The range has two commits and neither carries a `Pattern-Departure:` trailer. The directive declares no PATTERN changes. | Yes. |
| Stencils | None changed. | Yes. |

**Findings**

- **F1, LOW, consistency, `internal/shuttleengine/waitmark.go:175-177`.**
  - Summary: the `shellWaitActive` doc says a gated run's shell-only waiting turn end "is judged at once, so neither leaves a list to show". That is now false. A gated run with every output file present now waits on such shells up to the bound, and `syncShellWait` reads `shellWaitActive` every tick.
  - Evidence: the code at `waitmark.go:178-194` is unchanged. The comment names the removed behaviour. `wait.go:1124-1130` shows the gated path waiting.
  - Fix: reword the sentence to say a gated run with every output file present counts Done in the same tick the last shell crosses the bound, so a staggered list can still show while an earlier shell is past the bound and a later one is not. Keep the comment's claim about when the mark shows consistent with that. This also decides whether the marker behaviour is now reachable for gated runs.

- **F2, LOW, consistency, `internal/shuttleengine/spec.go:124`.**
  - Summary: the `AwaitedShellPrefixes` comment says an awaited shell "keeps a gated waiting turn end from being judged as a plain turn end". The gated judgment for non-awaited shells is no longer "plain" when every output file is present.
  - Evidence: `spec.go:124`, against `wait.go:912-927`.
  - Fix: reword to say an awaited shell keeps a gated waiting turn end waiting with no nudge and no expiry, where a non-awaited shell is nudged and expires into Done at the bound.

- **F3, LOW, consistency, `internal/websterengine/runlevel.go:39`.**
  - Summary: the comment says a gated turn end waiting only on non-awaited shells "is judged as a plain turn end". That holds only when an output file is missing.
  - Evidence: `runlevel.go:39`, against `websterengine/doc.go:640-643`, which the diff updated.
  - Fix: change the clause to "which one waiting only on non-awaited shells is held when an output file is missing and is not counted done inside the shell wait bound".

- **F4, LOW, consistency, `internal/shuttleengine/claudeengine/testdata/corpus/master-fork-running/case.yaml:12`.**
  - Summary: the case description says a gated run "judges a turn end that waits only on a shell no awaited prefix names as a plain turn end". This is the same stale claim, now false for a run with its outputs present.
  - Evidence: `case.yaml:12`, with line 43 repeating "held at once as a plain turn end with an output missing". The fixture's own case has an output missing, so line 43 is correct. Only line 12 is general.
  - Fix: qualify line 12 with "when an output file is missing". Leave the fixture's expected outcomes unchanged.

**Tool calls (1)**
1. `grep -rnE` over Go, MD and YAML files for "plain turn end", "judged at once" and `background_shell_wait_min`. My context did not hold the other readers of the old gated wording, such as `spec.go`, `waitmark.go`, `runlevel.go` and the corpus fixture.
