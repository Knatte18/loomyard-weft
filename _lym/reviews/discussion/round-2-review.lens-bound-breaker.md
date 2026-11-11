```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked every bound, guard, guarantee and claim about existing code that the Sketch states.

**Claims attacked**
- D1, the lock covers every Go writer of the record past approval: held. Darn-Write's guard restores nothing itself (`internal/loomshed/darnwrite.go:229-252`), and Record-Write never overwrites a record.
- D1, the base hash under the lock makes the check and the write atomic: held. The lock order is the decision lock, then the weft write lock, then the board lock, and no path takes them in reverse.
- D1, refusing at Webster rows keeps a folded entry in Merriam's view: **broken** (F1).
- D1, the commit-failure branches told apart by this call's trailer: held. A trailer that matches an earlier identical replace leaves the same end state.
- D1, the bound "replace can drop added entries and bullets…; a replace that only drops added entries changes no reviewed text": **broken** (F5).
- D2, the cut point is the newest version whose bytes match the trailer's sha256: **broken** (F2).
- D2, "Only a lym verb writes a commit message on the records branch": **broken** (F3).
- D2, a trailer for another record's path does not bound this record: held.
- D3, the board copy applies when the entry carries a stored record: **broken** for a re-entered run before approval (F4).
- D3, the refusal once the filing is done (`raddleRunFiling`): **broken** in the crash window between saving `board.json` and writing the store (F7).
- D4, the run lock, worktree check and caller check as bounds: the caller check is **broken** (F6). The run lock and worktree check hold, within the stated window.
- D4 and the Open risks, the adopted live weft: **broken** (F8).
- D4, teardown leaves the branch live only while the pair stands or after a failed remote step: **broken** (F9).
- D4, Record-Write never overwrites a held record file: held (`internal/loomshed/recordwrite.go:124`).
- D5, the listed writers stamp the hash, and replay carries it: held, except for mixed binaries (F10).
- D5, the Sketch conversion, label edits and promote leave the body alone: held (`endsAsSketch`, `internal/boardengine/store.go:796-838`).
- D6, the guidance to fold every added entry into the main decisions: **broken** on Record-Write and drift runs (F11).

**Findings**

F1 — MEDIUM, design. D1, the Webster-row refusal.
- A fold made between plan approval and Webster removes a decision from Merriam's view.
- Counterexample: after plan approval, the parent records `add` without `--amends-plan` at a Drift row before Webster. Then a `replace` folds that entry into a D-entry. Webster starts.
- Master renders only the record's `Added after Discussion` entries into `parent_decisions`, and the plan was never amended, so the decision reaches no one.
- The same holds for a run sent back into Webster by a goto after a fold.
- Evidence: `internal/websterengine/doc.go:340-342`.
- The refusal only covers standing at a Webster row. It needs to cover every row from plan approval through Webster, or the fold has to keep the entry visible to Master.

F2 — MEDIUM, design. D2, the cut point at the newest matching version.
- Counterexample: a replace writes bytes X. Then `add` appends F, giving Y. A writer deletes F by hand, which leaves exactly X again, and a sweep commit (`lym fabric commit` or a discussion commit) lands it.
- The versions are now X (sweep), Y, X (replace). The newest version matching the trailer's hash is the sweep commit, so the cut leaves only [X] and the drop of F is never flagged.
- This breaks the acceptance criterion "D2: an entry added after a replace and then dropped by a writer is still an `added-decision-dropped` finding" whenever the drop is committed.
- `add`'s `checkDarnRecord` would then also admit the record.
- Fix: cut at the newest matching version at or before the trailer commit.
- Evidence: `internal/discussionparser/append.go:238`, `internal/loomcli/decision.go:470-476`.

F3 — MEDIUM, design. D2's bound, "Only a lym verb writes a commit message on the records branch".
- No guard enforces this: `internal/shellguard` holds no `git commit` deny (a grep found none).
- An agent session, or the operator, can commit in `_lym` with a forged `Lym-Record-Replaced: parent <path> <sha>` trailer over a record that dropped entries.
- That silences the very history check the trailer bounds. This is the same forgery the support log rejected the content marker for.
- Evidence: `_lym/discussion/support-log.md:68`; the stencil's "no git commit" is prose only (`loom-template-discussion.md:156`).

F4 — MEDIUM, design. D3's second condition, the entry carrying a stored record.
- Counterexample: a tapestry run past approval is sent back with `lym loom goto --to Discussion-Write`. During the new Discussion-Review, or the Discussion-Bouncer settling window, the parent runs `add`.
- The entry already carries a stored record, so D3 copies the unapproved, in-review record onto the board.
- That breaks the invariant in `decision.go`'s header that a call made while Discussion-Bouncer is settling is not copied to the board.
- Evidence: `internal/loomcli/decision.go:5-6`, `:534-537`.
- The condition needs to be limited to recipes with no Discussion approval row.

F5 — LOW, design. D1's bound.
- The bound names only dropping entries and bullets. The file may also rewrite an added entry's body, or insert a new `### Added after Discussion (…)` entry.
- `IdentityText` strips both, so neither triggers a re-review.
- The reverse also fails: dropping an appended PATTERN bullet, which `IdentityText` keeps, does change reviewed text.
- Evidence: `internal/discussionparser/identity.go:11-18`.

F6 — LOW, design. D4's bound "to callers that are not a run's agent".
- `isRunAgentSession` reads the strand name from the environment. An agent that unsets the variable passes as the operator's shell.
- `decision.go` states this limit for its own guard ("guards against a mistaken call, not a forged strand name"). D4's bound claims more than its guard gives.
- Evidence: `internal/boardengine/taskguard.go:14-17`, `internal/loomcli/decision.go:9`.

F7 — LOW, design. D3's filed refusal.
- `FileRaddle` saves `board.json` (run entry turned into the project) before writing the store; `store.json` is written last.
- A crash in between leaves `raddleRunFiling` reporting "not filed". `add` or `replace` in that window copies the run's record over the project's own record, which is exactly what D3 exists to prevent.
- Evidence: `internal/boardengine/raddlefile.go:146-150`, `:337-361`; `internal/loomcli/raddlewiring.go:82-97`.

F8 — MEDIUM, design. D4 and the fourth Open risk, the adopted live weft.
- When a raddle run adopts the motif pair's live weft branch, the worktree already holds `_lym/discussion/decision-record.md` from the motif run.
- Record-Write never overwrites a held file, so the raddle run reads the stale motif record. The record the operator set with `--record-file` never reaches it.
- The Open risk instead assumes the working record is the folded one, so it misstates the failure and its fix ("a `replace` whose file is the record's own bytes").
- Evidence: `internal/loomshed/recordwrite.go:120-124`, `:136-173`; `internal/loomcli/projectrecord.go:51-57` (motif and raddle share the record path).

F9 — LOW, design. D4's claim about when the branch stays live.
- The teardown's remote step is a detached child: it pushes the tip and the tag, then deletes on origin.
- Until the child finishes, origin holds the weft branch with no `archive/<slug>/*` tag covering its tip, so `Topology.Add` adopts it.
- So the branch is live during that in-flight window too, not only while the pair stands or after a failed remote step.
- Evidence: `pattern/PATTERN-fabric-boundary.md`, "Unwire and teardown"; `internal/fabricengine/add.go:151`.

F10 — LOW, design. D5's stamping.
- An older binary on another machine keeps `record_body_sha256` as an unknown key, but stores a record without restamping it.
- After that, the hash describes whatever body the last new-binary write saw, so promote warns spuriously or misses a stale record.
- Mixed binaries are the case `PATTERN-tolerant-records` exists for, and D5 does not state this bound.
- Evidence: `internal/boardengine/decision.go:44-63`, `pattern/PATTERN-tolerant-records.md`.

F11 — MEDIUM, design. D6, "folding every added entry into the main decisions".
- Several readers give `Added after Discussion` entries precedence over the rest of the record, and a fold drops that precedence:
  - Darn's scope is "the board entry's body under `## Goal`, with the record's `Added after Discussion` entries over it".
  - Darn's gate re-prompt lets only such an entry override a finding.
  - Drift's seats say these entries "win over the rest where they differ".
- On a Record-Write record, the Goal holds the fenced body. A folded D-entry that contradicts it has no precedence, and the darn writer may not read it as scope at all.
- Evidence: `internal/darnengine/doc.go:64`, `internal/loomshed/darnwrite.go:47-56`, `internal/driftengine/seats.go:104,113`.

Tool calls: 1. I read `internal/loomshed/darnwrite.go` (lines 15-60 and 215-260) to see whether Darn-Write's guard writes the record, a Go writer one hop past the cited ones that my context did not hold.
