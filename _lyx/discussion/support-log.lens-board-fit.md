```yaml
lens: board-fit
outcome: reported
```

Lens board-fit: the record checked against every other board entry, for duplicated or contradicted planned work and dependencies it leaves unnamed.

**MEDIUM — Scope / D9 / orch role stencil: overlap with the running task `discussion-scope-in-lenses` is not named.**
- Both tasks edit the orch role stencil and the landing texts.
  This record drops the start steps and the `landed` notice handling.
  That task removes the parent-review gate and the `lyx loom review` verbs, and renames `Merge-In` to `Parent-Align`.
- Both are running now.
  The record names no ordering and no merge risk for the shared files: `orch-template-role.md`, the batten notice texts, and the landing-halt wording in `arbiterstop` and `sheddriver` that names `Merge-In`.
- Evidence: the `discussion-scope-in-lenses` entry, status `running · Worktree-Create`, Part 3 rename scope "the recipe row …, the stencils and docs".

**MEDIUM — D8: note `landing-escalate-arbiter` is not named.**
- D8 rewrites `arbiterstop` eligibility per recipe and keeps landing halts as hand-backs for batten.
- The note asks for a Merge-In or Finalize fixer `escalate` to reach the arbiter first, in that same function.
- The record neither folds the note in nor states that it leaves it untouched.
  D8's "Loom's and darn's rules are unchanged" makes the note's later change collide with the new per-recipe structure.
- Evidence: the `landing-escalate-arbiter` body, "`internal/arbiterstop` marks it `LandingHalt` and skips the arbiter".

**MEDIUM — D4 / Open risks: process volume, given note `spawn-audit-log`.**
- D4 spawns one `lyx batten run` pass per run per `notice_probe_s`, plus one per status write.
  Each pass runs its own git, reed and process reads.
- The board records a machine freeze from a process storm of about 800 starts a second with four runs in flight.
- The record lists process volume only as an open risk, with no bound in D4 on passes a minute per dobby, or hub-wide.
- Evidence: the `spawn-audit-log` "Seen" section.

**MEDIUM — D4: `battencli.ResumeInWindow` callers are left unaddressed. This is board-adjacent rather than a pure board conflict.**
- `lyx adopt`'s step 5 resumes every adopted run through `battencli.ResumeInWindow`.
- `lyx beam land` starts the beam's landing run the same way.
- D4 retires `--window` and the long-waiting batten process, but does not say what these two paths become.
  Without that, adopt and beam land either start a long-lived batten that D4 says no longer exists, or start nothing for dobby to see until its next tick.
- The `adopt-runs-tests` note also plans more work in adopt.
- Evidence: `internal/adoptcli/doc.go` "Step 5 resumes each adopted task … through `battencli.ResumeInWindow`", and the `beamcli` doc's land section.

**LOW — D2 / D3: `loom-beam-darns` step 4 turns a running task's worktree into a beam while the task's run continues.**
- `lyx beam promote`, which D2 extends, ends that run for good.
- The record is right for today, but it says nothing about whether dobby's ownership rule (D3) fits a beam whose own task run is still in flight.
  That case is the darns note's main open question: "What drives the darns inside the beam: the beam's daemon".
- Naming `loom-beam-darns` as downstream work would avoid a silent contradiction later.

**LOW — D9: `creel-agents` plans for drivers' stop reports to post instead of duplicating batten's notices.**
- D9 keeps batten's halt notices, so the two designs overlap on the same notices.
- No contradiction now, since `creel-agents` is a note, but the relation is unnamed.

**LOW — D5: README draft mark.**
- Note `board-viewer` plans a README restructure: collapsible sections and grouping by type and area.
- D5's "`draft` before its title" mark should survive that restructure; the record doesn't mention it.

**LOW — Scope: `batten-window-default` removal is consistent and allowed, as a note write.**
- The `idle-notice-dead-waiter-cause` note reads InnerRun's agent-activity path, which D4 moves into passes.
- Whether the dead-waiter Warn still fires once per episode across passes depends on the D4 memory record.
- That note is worth naming as touched.

Calls I made beyond my context:
1. `lyx board get` for `creel-agents`, `spawn-audit-log`, `fabric-remote-async`, `idle-notice-dead-waiter-cause`, `reed-status-buttons` and `adopt-runs-tests`: the writer had not read these entries, and board-fit needs every entry that might touch the record.
2. A grep of `internal/adoptcli/doc.go` for resume and window: the writer never read adopt, which another verb uses to resume runs.
