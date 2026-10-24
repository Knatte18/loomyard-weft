```yaml
lens: fix-claims
outcome: reported
```

Task: check that each claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

**MEDIUM: D11's new Bound sentence (decision-record.md lines 196–197), carried over from the review's F6 proposal**
- Changed text: "it is bounded by the `finalize` gate over the merged tree and by the fixer's size and kind caps, past which Finalize halts for the orch."
- Summary: the fixer's size and kind caps never measure the conflict session's resolution, so they do not bound what this Bound says they bound.
- Evidence:
  - `internal/landingshed/doc.go:43`: the fixer's change is judged by `LineChange(Baseline, HEAD)`, and Baseline is the HEAD before the fixer's first turn.
  - `internal/landingshed/doc.go:33`: in a darn Finalize the conflict is resolved by the resolver's conflict session, under the lock, before the gate and fixer run.
  - So the resolution is already in Baseline and stays outside the caps. Only the `finalize` gate bounds it.
  - The caps bound the fixer's own later commits. `doc.go:50` says this: "The size and kind caps bound what an unreviewed Finalize fix lets through".
  - The sentence needs to say that the gate alone bounds the resolution, and the caps bound only the fixer's change on top of it.

The other claims I checked hold:
- **D11 left-pair refusal:** the three parts it names (task worktree, records worktree, records branch) match `leftPartNames` in `internal/battencli/startguard.go`.
- **D8:** the landing's merge fixer and conflict session do work in the beam worktree (`doc.go:33`, `:43`).
- **D11:** "past which Finalize halts for the orch" holds for a recipe that declares no reroute review (`doc.go:45`, `:105`).

Tool calls:
1. `sed`/`grep` on `internal/battencli/startguard.go:100-110` and `internal/landingshed/doc.go`: my context had only part of `leftPartNames` and none of the landingshed doc's fixer-cap text.
2. `grep` for "conflict session" and "resolver" in `internal/landingshed/doc.go`: to find where the conflict resolution sits relative to the fixer's Baseline.
