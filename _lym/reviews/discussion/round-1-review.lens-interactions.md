```yaml
lens: interactions
outcome: reported
```

Lens: interactions. Seven pairs are broken: one BLOCKING, four MEDIUM, two LOW. The rest held.

**Pairs checked**
- D4×D5 (records commit before merge, one write lock): held, within D5's stated bound.
- D4×D6 (Warp-SHA meaning × `OriginCodeTip`): held while there is no replay.
- D4×D7 (Warp-SHA rewritten on replay): broken, F2.
- D4×D9 (records commits move HEAD × SHA readers): held at this lens. The webster check in `reconcileHead` allows only merge commits between a fork's reported head and HEAD; that path is outside this lens and is left to existing-paths.
- D5×D7 (merge trailer × prime push guard): broken, F5.
- D5×D8 (dirtiness guards commit records first × fast-forward of the local parent after a GitHub merge): broken, F1.
- D5×D9 (strip merge × held-work check): held.
- D5 anchor change × D7 and D9 (which anchor defines the records paths): broken, F4.
- D6×D7 (owed prime push × failed-push record and act rule): broken, F7.
- D6×D8 (teardown deletes the view × refresh on push and GitHub deleting the head): held.
- D7×D11 (replay "on any branch" × Bolt by-entry replay of `lym-bolt`): broken, F3.
- D1×D2 (layout read from the marker × clone of an origin holding `lym-bolt`): broken, F6.
- D1×D3×D5 (marker tracked, never excluded, restored to the receiver): held.
- D3×D9 (shared exclude × `DirtyPaths`): held.
- D3×D11 (shared `.git/info/exclude` reaching `_bolt`): held.
- D8×D9 (view merged on GitHub × landed judgment by held work): held.
- D8×D7 (view force-push × prime guard): held, since the guard binds records pushes only.
- D10×D3: held.
- Constraints vs D1, D2, D3, D8 (exclude rule, cwd resolution, fabric boundary, remote-refs lock, named constants): held.
- Acceptance criteria vs the decisions above: none contradicts a decision. None covers F1, F4 or F5.

**Findings**

- **F1 — BLOCKING, design, decision-record.md D8 (with D5).**
  - Summary: after GitHub merges the view, D8 fast-forwards the local parent to origin's parent tip. D5 has every fabric dirtiness guard, `FastForwardPair`'s included, commit the worktree's records first. That new local records commit, like any unpushed records-only commit D7 leaves on the parent, means local HEAD is no longer an ancestor of upstream, so `FastForwardPair` refuses with `FastForwardNotFastForward`. The prime and every beam always hold live records, so Finalize cannot complete a pull-request landing.
  - Evidence: `internal/fabricengine/parenttips.go:306-331`, where the dirty check runs and then `IsAncestor(localHEAD, upstreamSHA)` refuses; `internal/landingshed/finalize_pr.go:406-416`, where `mergeOnGitHub` fast-forwards the parent pair today.
  - Fix: D8 brings the local parent level with origin through D7's records-only replay onto origin's tip, not a fast-forward. Its acceptance criterion should run with uncommitted and unpushed parent records.

- **F2 — MEDIUM, design, decision-record.md D7 (with D4).**
  - Summary: D7 rewrites each replayed commit's `Warp-SHA` to its new parent. When origin's diverging commits carry code, a replayed transition commit then names a code tip the transition never judged. `lastTransitionCodeTip` and `OriginCodeTip` would hand over or settle on that tip.
  - Evidence: `internal/fabricengine/handover.go:336-346`, where the tip is the transition commit's `Warp-SHA`; `internal/fabricengine/codetipsettle.go:31-37`.
  - Fix: state the rule. Either rewrite only when origin's new commits are records-only, or make the replay of a transition commit over foreign code stop as owed.

- **F3 — MEDIUM, consistency, decision-record.md D7 vs D11.**
  - Summary: D7 says "every replay … on any branch" replays only records-only commits, judged by `&lt;anchor&gt;/_lym` paths. Board commits on `lym-bolt` touch no such path, and the Bolt is replayed by entry through the Bolt replayers. Read literally, D7 turns every Bolt pull with a local board commit into an owed stop, which contradicts D11's "as today".
  - Evidence: `internal/fabricengine/replay.go:4-5`.
  - Fix: scope D7 to code branches and state that `lym-bolt` keeps the by-entry Bolt replay.

- **F4 — MEDIUM, design, decision-record.md D5 anchor change vs D7 and D9.**
  - Summary: an anchor change is a records commit that moves `_lym` from the old anchor to the new one. D9 and D7 define the records paths from one anchor, a commit's own marker. The deletions under `&lt;old anchor&gt;/_lym` then read as code changes. Three things follow:
    - the verify and green tree identity change;
    - a webster card check accepts the records commit as code work;
    - D7 calls it not records-only and refuses to replay it.
  - Evidence: `internal/fabricengine/anchormarker.go:93-118`, where `committedMarker` and `treeAnchor` read one marker per revision.
  - Fix: state that a commit's records paths are the union of its parent's anchor and its own, and add an acceptance row for an anchor-change commit.

- **F5 — MEDIUM, design, decision-record.md D7 (with D5).**
  - Summary: the prime guard admits a commit "carrying fabric's merge trailer", but a non-squash D5 landing also brings in the task's code commits as second-parent ancestors. Those are commits origin lacks on the prime's branch, are not records-only and carry no trailer. Two readings are possible, and the record picks neither:
    - every prime records push is owed until the landing's own parent push lands;
    - "origin lacks" means not reachable from any origin ref.
  - Evidence: `decision-record.md:125-130` against D5 steps 1-4 at `decision-record.md:91-96`.
  - Fix: define "origin lacks", for example as not reachable from any origin ref, and state how a records push between a landing's merge and its parent push behaves.

- **F6 — LOW, design, decision-record.md D2 vs D1.**
  - Summary: D2's clone path for an origin holding `lym-bolt` writes no marker. If origin's default branch lacks `.lym-anchor`, for instance because the default branch changed after the hub was made, D1 reads the prime as two-repository while D2 built a one-repository hub.
  - Evidence: `decision-record.md:32-35` and `decision-record.md:45`; `internal/lymcwd/anchor.go:176-190`, where a missing own marker falls back without error.
  - Fix: D2 refuses that clone, naming the missing marker, or commits the marker as records.

- **F7 — LOW, consistency, decision-record.md D7 vs D6.**
  - Summary: D7 says a run in the prime "waits at its transition" on an owed prime push. The act rule counts a failed-push record as owed only for classes transport, deferred or teardown_lock whose `Runs` name the slug. D7 adds a guard refusal without naming its class or `Runs`, so a new class would not hold the run, contrary to D7.
  - Evidence: `internal/fabricengine/failedpush.go:272-293` (`FailedPushesOwedBy`).
  - Fix: D7 names the class the guard's refusal records and the runs it lists.

**Tool calls**
1. `internal/fabricengine/failedpush.go` function and field list: my context lacked how failed-push records are keyed.
2. `internal/fabricengine/failedpush.go:214-297`: my context lacked how an owed push is matched to a run (`FailedPushesOwedBy`, `pairSlug`).
