---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:9
    summary: The record designs from a HEAD that main has overtaken; main and prod already carry a52467411, Move the Go driver and tackler into shed; batten drives through it, and the live hub config already holds shed.yaml and inheriting overrides.
    fix: Before any further discussion round, take main into the task (the Merge-In route) and rewrite the record against the merged tree. Add a Constraints bullet naming commit a52467411 as already landed, and cut every Decision it already delivers. Rescope Goal and Scope to what remains, or have the record say the task is spent and hand back to the parent. Rewrite D5's Migrate paragraph and acceptance line 166 against the hub's live loom.yaml, which already holds the inherit values. Correct support-log.md Turn 1's third finding, which says the hub loom.yaml spells out the defaults. Reconcile Scope Out's last bullet, no default turns to go, with the hub shed.yaml's driver_kind go.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:74
    summary: D4 names the agent settings' denies as a bound on a batten tackler's writes in the prime, but no installed deny covers Edit, Write or git commands, and the bound that does exist, batten never pushing the prime, goes unstated.
    fix: In D4 line 74 and its Bound line (line 79), drop the agent settings' denies as a bound. State the bounds that hold. The stencil's write rule is a prompt rule only. Batten never pushes the prime (internal/battenshed/doc.go), so a tackler commit stays local even when a resume re-enters Land. Name what a resume into Land or Worktree-Teardown does with a dirty prime working tree or a local commit. In acceptance line 169, replace the clause about the same denies as a loom tackler with a checkable case. For example, a batten resume into Land over a local commit the tackler made in the prime does not push it.
---

### [BLOCKING] The task's design has already landed on main

**Location:** `_lyx/discussion/decision-record.md:9` (Scope), with knock-on at lines 24, 94–97, 166 and `support-log.md:16`

**Issue:** `git branch -a --contains a52467411` lists `main`, `origin/main` and `origin/prod`. That commit's subject is "Move the Go driver and tackler into shed; batten drives through it", dated 2026-10-15 10:16, after this worktree's HEAD `9ea1f7474` (07:31 the same day), and it is not an ancestor of HEAD.
The hub board's config commit `c2dba0788`, "reconcile hub-wide config for build (a52467411ace production …)", added `_board/_lyx/config/shed.yaml` with `driver_kind: go`, `tackler: haiku`, 10 and 4.
It also gave `loom.yaml`, `darn.yaml` and `batten.yaml` the inherit values D5 and acceptance line 167 describe: `""`, `0` and `~`, with comments naming `shed.yaml`.
I read only commit metadata and the live hub config, never main's code, as this round's code-base rule requires.
That evidence is enough to show three things.
The record's premise that this work is still to be built is false.
D5's one-shot Migrate (lines 94–97) has nothing to match on this hub, and support-log Turn 1's finding that the hub `loom.yaml` spells out every default no longer holds.
Scope Out's last bullet ("every default stays `llm`") contradicts the hub's live `shed.yaml`.
A plan written from this record would re-implement landed code and collide with it at merge.

### [MEDIUM] D4's bound on a batten tackler's prime writes names a bound that does not exist

**Location:** `_lyx/discussion/decision-record.md:74`, `:79`, `:169`

**Issue:** D4 states what a batten tackler without a tamper snapshot can let through: writes, index changes and a local commit in the prime, followed by a resume that re-enters Land or Worktree-Teardown.
That much is stated, as the focus asks.
The stated bound is "the tackler stencil's write rule and the agent settings' denies".
The stencil rule (`contracts/stencils/loom/loom-template-tackler.md:36-44`: write only the verdict and case brief, run no git command that changes anything) is prose for the model, not a mechanism.
The denies `internal/shuttleengine/claudeengine/settings.go` installs cover the Agent tool, AskUserQuestion, Python, in-place `sed`/`perl`, `rm` on a computed path and the machine sleep commands.
None of them refuses Edit, Write, `git add` or `git commit`, so the denies bound none of the writes D4 lists.
Acceptance line 169 checks only that the batten tackler gets the same denies as a loom tackler.
That is parity with a set that bounds nothing here, so it confirms the overstated bound and tests no real one.
The stop-key check bounds run movement, not writes, as D4 itself says.
A real mechanical limit does exist: `internal/battenshed/doc.go:160` states batten never pushes the prime, so a stray tackler commit stays local when Land is re-entered.
D4 does not name it, and does not say what Land's pull or Worktree-Teardown does over a dirty prime.
The wrong bound came from round 1's F1 fix, which offered "the stencil's write rule plus the agent settings' denies" as one option, and the fixer took that wording.
This is a correction of that wording, not a regression.

## Focus verdicts

- D4 tamper bound: the let-through is stated, but the named bound is wrong (F2), and the Bound line and acceptance line 169 repeat the same wrong bound.
- D5 Migrate scope: not a defect at HEAD.
  The record scopes the rewrite to a hub `loom.yaml` holding all four old defaults with their template comments.
  `configreg.PrepareRunSettings` (`internal/configreg/runsettings.go:56-62`) builds each `run_settings` document with `yaml.Marshal`, so the document carries no comments and the comment marker cannot match it.
  Line 93 ships the inherit value in all three templates, and acceptance lines 166–167 cover the chosen scope.
  F1 makes the paragraph moot on this hub.
- Per-recipe overrides across D3, D5 and D6: not a defect.
  D5 line 91 selects every tackler key from the seed recipe's module at `lyx loom drive`'s wiring, and from `batten.yaml` for batten's own run.
  D6 selects `driver_kind` at both seeding sites by the child's recipe.
  Every added key has a reader: `darn.yaml`'s four through drive and the seeding sites, and `batten.yaml`'s three through batten's own run.
  `batten.yaml` carries no `driver_kind`, and acceptance line 164 refuses one there.
- Halted-row-only resume across D2 and D4: not a defect.
  Today's check (`internal/loomcli/tackler.go:359-372`) accepts the current row or the routed-from row.
  D2 line 49 turns the allowed set into a recipe seam, and batten tells the current row only.
  Acceptance line 168 checks that a routed-from goto is refused and hands back.
