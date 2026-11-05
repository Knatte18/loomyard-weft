---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md D11, D10, D4, D6 (Bound)
    summary: "The records that carry containment state (amendments, deferral, scope off, last passing check head in scope.json; stored matches rulings in judged.json) are plain tracked files the writer can edit, and no Decision states a bound; D6's Bound claims a forged seal needs a hand-made weft commit, which an edit of scope.json makes false."
    fix: "In decision-record.md extend D6's trailer mechanism to the two mutable records. (1) D11, D10 and D4: every write of scope.json's check result, amendment, deferral and session fields and of judged.json (by `lym drift amend`, `lym drift defer`, Drift-Baseline, Drift-Expect and Drift-Verify) is committed through D6's fabric seams in one weft commit carrying a `Drift-Records: <sha256 over scope.json and judged.json>` trailer. (2) D6's seal-check paragraph gains a records check: the newest weft commit carrying `Drift-Records` holds the copies, the current files must hash to its trailer, and every reader (Drift-Verify, Drift-Recheck, the `drift` gate entry, `lym drift check`) takes the scope, deferral, amendments, last-passing-check head and stored rulings from that commit's copies. (3) A mismatch is reported like a seal failure: a Stuck at a row naming the parent's `lym drift amend --restore`, which in this case writes both files back from the commit, and a report without failure at the gate entry and `lym drift check`. (4) Rewrite D6's Bound to state the true bound, that a forged seal or forged record needs a hand-made weft commit with the trailer, and drop 'no agent's verbs make' as the only bound. (5) Add the records check to the acceptance bullets of D6, D10 and D11, such as an edited amendment in scope.json or a hand-added stored ruling in judged.json stops Drift-Verify naming `--restore`."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md D9
    summary: "The judge tree guard does not cover ignored paths, keeps its snapshot in a path the judge can write, and offers no restore for a modified or deleted untracked file or for a snapshot lost with .lym."
    fix: "In D9 rewrite the snapshot sentences. (1) State that the snapshot covers, in both checkouts, HEAD, the tracked tree, every untracked path outside ignore rules with its content kept as a copy beside the snapshot, and, for the ignored `.lym/drift/` tree, a path to content-hash list of the baseline directory and of the current capture's scratch directory the judge reads added and removed case files from. (2) Store the snapshot in the hub's board-level `.lym` beside the run process record, under a name `internal/lymdirs` declares, not in `.lym/drift/judge-snapshot.json`: the worktree's `.lym` is a fabric-wired link into the weft-side `.lym` (internal/fabricengine/gitexclude.go), so the judge session writes there and the record's 'outside both trees' is false. (3) Extend the way forward to name a modified or deleted untracked path (restore from the snapshot's copy) and a changed scratch capture directory (delete it and `lym loom resume`; a baseline change is F3's). (4) State what a resume does when the snapshot is gone, such as `.lym` cleaned or another machine: Drift-Verify takes a new snapshot before the respawn and the Stuck for the earlier session is dropped, since no evidence of its edits remains. (5) Update D9's acceptance bullet to match."
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lym/discussion/decision-record.md D9 (Decision, last guard sentence) and D9 acceptance bullet
    summary: "D9 says a judge-made baseline change 'stops as D7 says' (a digest mismatch naming `lym drift defer`), but the D9 acceptance bullet says it is a Stuck naming the changed paths and the restore; a single digest names no path and nothing holds a copy to restore, and D9 does not say the digest is recomputed after the judge returns."
    fix: "In D9 replace 'a changed baseline stops as D7 says' with: after the judge session returns and before Drift-Verify closes the judge's mark, it recomputes the baseline digest (D7); a mismatch is a Stuck naming the parent's `lym drift defer` (D12), the way forward D7 names. In the D9 acceptance bullet drop 'to the baseline' from the list that names changed paths and the restore, and add: a baseline change made by the judge session stops Drift-Verify naming `lym drift defer`."
---

### [BLOCKING] Mutable drift records are an unbounded bypass of the seal and the judge

**Location:** `_lym/discussion/decision-record.md` D11, D10, D4, D6 (Bound)

**Issue:** The record's own threat model is in D7's rationale: a writer handed an unexpected cell's old and new value could edit the baseline to match, so a digest in a weft commit made before the writer turns the edit into a stop.
D6 does the same for `expected.yaml` and states "forging a seal needs a hand-made weft commit carrying the trailer before the writer's first warp commit".
The decisions then keep the state that decides containment in files with no such protection.
D11 stores every amendment in `scope.json` and says "Containment reads the sealed expectation, or an empty one after a `--discard`, and every amendment".
D4 makes a row "Done at once" when the recorded scope is `off`, when the capture was deferred, or "when the warp head equals the head of the last passing check", all read from `scope.json`.
D10 reuses a stored ruling for a reproduced tuple without a judge, read from `judged.json`.
Both files are tracked under `_lym/drift/`, and the support log's Turn 2 records that "no guard keeps an agent from editing `_lym/drift/`".
So a writer that adds an amendment entry by hand to `scope.json`, sets the last passing check to the current head, or adds a `matches` tuple to `judged.json` passes Drift-Verify and Drift-Recheck without the expectation, the judge or the seal ever being consulted, and no check in the record notices.
D9's judge guard names `scope.json` and `judged.json` as judge-protected only in the acceptance bullet; the writer's row has no equal guard.
D6's Bound is therefore false as written, and D11's Bound ("an amendment only adds coverage ... it cannot remove one entry or change the seal") bounds only the verb, never an edit of the file.
The attack-surface rule requires the Decision that introduces such an edge to state what it can skip and what bounds it; neither does.
`lym drift defer` is already bounded as the parent's call, so the same bypass by hand-setting `Deferred` or `scope: off` is the larger exposure.
I checked the weft side: `internal/fabricengine/trailer.go` does append `Warp-SHA` to every fabric weft commit, so a trailer-carrying commit made by lym's own verbs is a workable anchor for these records, as it is for the seal.

### [MEDIUM] The judge tree guard leaves ignored paths, its own snapshot and several restores uncovered

**Location:** `_lym/discussion/decision-record.md` D9

**Issue:** D9 snapshots "each checkout's HEAD, tracked tree and untracked paths, warp and weft alike" and stores it in `.lym/drift/judge-snapshot.json`, "outside both trees".
Three gaps follow.
First, git ignores `.lym`, and `internal/fabricengine/gitexclude.go` describes the worktree's `.lym` as a fabric-wired link into the weft-side `.lym` that agents write through, so the snapshot file sits where the judge session writes and a judge can rewrite it, and the "outside both trees" claim does not hold.
The record never says whether "untracked paths" includes ignored ones; the current capture's scratch directory under `.lym/drift/`, from which the judge reads an added or removed case's file, is ignored and so unguarded.
Second, the way forward restores "a changed tracked file from that HEAD" and deletes "an added untracked path", which leaves a modified or deleted untracked file with no source to restore from.
Third, a resumed row "compares against the recorded snapshot, never a new one", but the snapshot lives in the machine-bound `.lym`; D12 covers a lost baseline, not a lost snapshot, so a resume on another machine or after a `.lym` clean has no stated rule.
Weft files outside `_lym/drift/` are covered, since the weft tree is snapshotted whole.

### [MEDIUM] D9 contradicts itself on what a judge-made baseline change names

**Location:** `_lym/discussion/decision-record.md` D9 (Decision, last guard sentence) and D9 acceptance bullet

**Issue:** The Decision says "a changed baseline stops as D7 says", and D7 says a digest mismatch names the parent's `lym drift defer`, since the branch-point code cannot be captured again.
The D9 acceptance bullet lists "a change to the baseline that the judge session made" among the changes that are "a Stuck naming the changed paths and the restore".
A baseline digest is one hash, so it cannot name a path, and no record keeps a copy of the baseline to restore from.
D9 also never says the digest is recomputed after the judge returns; D7 recomputes it at the start of a check, before the judge runs, so an edit made during the session would only surface at the next check, and Drift-Recheck is Done at once at an unchanged head.

## Rejected

- D2, D5 and D11 under `command` mode: the per-case expectation, the parser that refuses `keys` and `lines` entries and checks no case name, and amendment validation by D2's rules do not rely on D1 identities; no finding.
- Resume at an unchanged warp head re-running a judge until it rules `matches`: `lines` mode stores `does not match` (D10), and `command` mode stores `does not match` and `unexpected` keyed by warp head and case (D2), so a resumed row stops without a judge; only an amendment, which is the parent's act, drops a stored ruling; no finding.
- D3 and D15 for `drift.diff` and `drift.key_patterns`: D3 records both in `scope.json` and every later row, the gate entry and `lym drift check` read the recorded values, with the in-flight rule for a record with none; no finding.
- Way forward at each Stuck: judge `does not match`, a reproduced stored ruling, a failed or timed-out current capture, a capture that never counted and a moved warp head each name theirs; an exhausted `judgement` or `expectation` entry names resume, which respawns the agent; no finding.
- Capture lock and gate slot order: D7 states slot first, or the caller's held slot, then the lock, and its claim that the main capture takes the lock before its slot today matches `internal/driftengine/capture.go` (`RunCapture` takes the lock, then `runStep` takes the slot); the waiter holds a slot idle, which D7's Bound states; no finding.
- In-flight runs: D4 ends a live old chair at the entry of Drift-Baseline and Drift-Verify, and Drift-Recheck runs the same engine; D3 and D14 cover `partial` with its `skip` list, D12 covers `WaivedAt`; no finding.
- A hub `gate.yaml` entry for `drift-verify`: D14 says `gates` is a per-name map whose names the operator owns, and `internal/gateslot/doc.go` says the same, with a key lym does not read loading unchanged; no finding.
