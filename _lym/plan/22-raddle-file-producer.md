# Card 22 — raddle-file-producer

**Create:**
- `plan:internal/loomshed#RaddleFileDeps` -> `type RaddleFileDeps struct`
- `plan:internal/loomshed#NewRaddleFile` -> `func NewRaddleFile(name string, deps RaddleFileDeps) shedengine.ShedProducer`
- `plan:internal/loomshed#raddleMaxReturns` -> `const raddleMaxReturns = 2`
- `plan:internal/loomshed#TestRaddleFile` -> `func TestRaddleFile(t *testing.T)`

**Edit:**
- `internal/loomshed#loomshedAllowedImports`

**Uses:**
- `plan:internal/loomshed#RaddleState`
- `plan:internal/loomshed#UpdateRaddleState`
- `plan:internal/loomshed#CheckRaddleProposal`
- `plan:internal/loomshed#RecheckRaddleBoard`
- `internal/raddleparser#Parse`
- `internal/raddleparser#Hash`
- `internal/discussionparser#SplicePart`
- `plan:internal/boardengine#RaddleFiling`
- `plan:internal/boardengine#RaddleEntry`
- `plan:internal/boardengine#ErrRaddleRecheck`
- `internal/loomshed/doc.go`

**Intent:** D9, D8's filing half and D7's hash match: Raddle-File, a Go row with no agent.
`RaddleFileDeps` carries the raddle paths (proposal, cold, approval, stage, part records, state), the decision record path, and the seams `Check` (D6 at filing, `CheckRaddleProposal(check, true)`), `Recheck` (`RecheckRaddleBoard`), `Entry` (the run's entry `created` stamp, module labels and own `project` field), `StoreDone(created, hash)` (whether the raddle store holds the Sketch's directory with that `created` and `hash`), `File(boardengine.RaddleFiling)`, `CreateBeam(project)`, `Release(slugs)`, `Notify(line)`, `EndEntry()` and `Commit()`.
`Call`, every step skipped when its result is there, so a retry after a crash finishes the steps it missed:
- It reads the proposal and its hash, and asks `StoreDone`; step 3 done skips the checks below, also after a filed part has landed and been pruned.
- Until then it runs `Check`, then in interactive mode (the recorded mode, never the key) matches the hash against the recorded approval, which a return clears, then runs `Recheck`; a failing check, a hash matching no approval (a proposal edited after approval) or a moved board is a return naming the failing checks, the mismatch or the moved entries; the `Landed` entries are named in the return's reason or the row's notice without failing.
- A return deletes the stage and part-record outputs of the failed attempt, raises `returns`, records the reason and clears the approval in one state update, and is `Stuck` without `Halt`, which `on_stuck` sends to Raddle-Write in a new session in the recorded mode.
- A failure when `returns` already equals `raddleMaxReturns` is `Stuck` with `Halt`, for the parent, naming the failures, that the run used its two returns, and the way forward: settle the cause, then `lym loom goto Raddle-Write`, which card 21 takes as a fresh entry (a new session, the approval cleared); the count is not reset, so each further failure halts for the parent again (D8's bound).
- Step 1 stages the store files into the stage directory (approved proposal, cold proposal, the Sketch's record, the entry's `created` stamp, the hash); step 2 writes `decision-<slug>.md` per filed entry by `discussionparser.SplicePart` (Goal and Scope rendered from the part's contract and packages, its decisions, risks, criteria and PATTERN changes), the project's record and the `tapestry` shape's one part's record being the Sketch's record whole; each output carries the hash beside it, and a step is skipped only when its outputs carry the hash of the proposal now approved, so outputs of an earlier proposal are rewritten; `Commit` runs after step 2.
  A `SplicePart` error is returned as the step's error, naming the part; the inputs that could cause one, a decision id, criterion or risk bullet, or PATTERN id the record lacks, are refused earlier in the same `Call` by card 12's `decision-unknown`, `criterion-unknown`, `risk-unknown` and `pattern-unknown` through `Check`, which reads the same decision record.
- Step 3 calls `File` with the filing built from the proposal, the records and the stage, at the Sketch's own `project` level, its `Check` re-running D8 inside the board write;
  the project's `RaddleEntry` carries the proposal's project `type` as its type label, each part's carries the part's own proposal `type`, and both carry the module labels the `Entry` seam returns (the Sketch's non-type labels), never the `sketch` label;
  each part's `RaddleEntry` body is rendered from the proposal and names the Sketch slug, the part's contract (takes, delivers, exports) and its packages with their sizes; an error matching `boardengine.ErrRaddleRecheck` is a return like the pre-check's, any other error is returned.
- Step 4, for `beam`, calls `CreateBeam(project)`; step 5, for `beam`, calls `Release` with the filed parts (the board write skips absent, done and released parts, so a part that landed and was pruned before a retry is skipped).
- Step 6 calls `Notify` once with what was filed and records `notice_sent`; step 7 calls `EndEntry`, the run's last write, and the row is Done.
- A failure of `CreateBeam`, `Release`, `Notify` or `EndEntry` is `Stuck` with `Halt` naming the step and the way forward `lym loom resume`, which re-runs the row from the first step not done.
A halt before step 7 leaves the run's entry open, so it shows like any run's halt and resumes through batten.
The allowlist `loomshedAllowedImports` in `internal/loomshed/seam_enforcement_test.go` gains `github.com/Knatte18/loomyard/internal/boardengine`, with a comment line: the filing type and the re-check error the producer's seams carry, no geometry resolved through it.
`TestRaddleFile` (tier0, recording fakes for every seam, temp paths) has rows: each shape files once with its steps in order and the entry ended after the notice; the part records written, the `tapestry` part's and the project's equal to the Sketch's record, a `tasks` part's holding its Goal, Scope and only its own decisions; in the `tasks` row each filed part's body names the Sketch slug, its contract and its packages with their sizes, and in the `RaddleFiling` the recording `File` seam receives each part's labels equal its type plus the Sketch's module labels; `beam` creates the beam before releasing, so no part is released before its project is a beam; re-runs after an interruption after each step repeat no step; an interruption after step 3 skips the checks by `StoreDone` even when a filed part was pruned; an interruption after step 5 with a released part pruned goes on to steps 6 and 7; a re-check failure before step 3 returns (outputs deleted, approval cleared, returns 1); `File` failing with `ErrRaddleRecheck` returns the same way, and with another error returns that error; a D6 failure returns naming the check; an edited proposal in interactive mode returns naming the mismatch; an unchanged proposal after a return is not filed until approved again; a third failure halts naming the goto; a failed `CreateBeam` halts naming the resume; after a return and a changed, re-approved proposal the stage and records are rewritten and the new proposal filed; a `depends_on` on a pruned `board_checked` entry passes and is named.

**ImpactSummary:** A new loomshed producer only; nothing builds it until card 23.
