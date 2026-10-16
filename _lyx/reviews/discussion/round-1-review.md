---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D5, third bound bullet)
    summary: D5 bounds the bypass edge by an operator watching the window, but nothing in the open path ensures a watcher, and lyx shuttle send can drive an opened strand unwatched
    fix: >-
      In decision-record.md D5, replace the bullet that begins "It is one interactive session per call in the caller's own worktree, in its own window an operator watches" with "It is one interactive session per call in the caller's own worktree, refused under a covering pause, unchanged from today. It takes no turn until text reaches its pane, typed by the operator in its window or sent by `lyx shuttle send <guid>`, which accepts a live reed strand with no run; a caller that drives it through `send` holds a session that runs every tool without a dialog and with no shuttle guard, and nothing but the explicit `--permission-mode bypass` on the open bounds that." Leave the other D5 bullets unchanged.
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md (Scope, In, docs bullet; Notes for the plan writer, first bullet)
    summary: The one comment D4 makes false, buildOpenCmd's no-permission-flag sentence, sits only in the optional Notes, and Scope's doc list also omits the Claude.OpenLaunch doc comment that lists the checks run before the session id is minted
    fix: >-
      In decision-record.md Scope, In, extend the docs bullet to read "The docs the change touches: the `open` command's help text, the `buildOpenCmd` and `Claude.OpenLaunch` doc comments, the `OpenSpec` and open-launch paragraphs of the `shuttleengine` and `claudeengine` package docs, and the shuttle row of `docs/overview.md`." Then delete the first bullet of Notes for the plan writer (the `buildOpenCmd` comment one), since Scope now carries it.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md (AC4)
    summary: AC4 requires the refusal to come before a session id is minted, which no public surface can observe; the strand and marker-directory conditions already pin the ordering
    fix: >-
      In decision-record.md AC4, change "before a session id is minted, a strand is added or the marker directory is created" to "before a strand is added or the marker directory is created". D3 keeps its own ordering sentence.
---

### [MEDIUM] D5's watched-window bound rests on an unverified premise

**Location:** `_lyx/discussion/decision-record.md`, D5, third bound bullet.
**Issue:** The bullet bounds the bypass session as running "in its own window an operator watches".
`Runner.Open` (`internal/shuttleengine/open.go`) adds the strand with `render.AnchorOwnWindow` in a detached tmux window and returns at once.
Nothing in the open path ensures that anyone is attached to that window.
`lyx shuttle send` (`internal/shuttlecli/send.go`) types a turn into "a shuttle run's or a live reed strand's" pane by guid, and `lyx shuttle open` prints that guid.
So the orch can open a bypass session and drive it without anyone watching.
That session then runs every tool without a dialog and without shuttle's run guards: no python deny, no computed-rm deny, no gate-slot deny.
The rubric's attack-surface test asks the Decision to state what the edge lets through and what bounds it.
Of the bounds D5 states, this one is not true in the code, and the real path, an unwatched session driven through `send`, goes unnamed.
The other D5 bullets match the code.
The open line passes no `--settings` (`buildOpenCmd`), a covering pause refuses before `OpenLaunch` runs, and the flag is opt-in.
The deny-rules-and-hooks-in-bypass bullet is marked as Claude Code behaviour, which is acceptable.

### [LOW] The comment D4 falsifies is only in the optional Notes

**Location:** `_lyx/discussion/decision-record.md`, Scope (In, docs bullet) and Notes for the plan writer.
**Issue:** `buildOpenCmd`'s doc comment (`internal/shuttleengine/claudeengine/command.go`) says the line passes "no permission flag".
It is the only text in the repository that the change makes false.
`docs/overview.md`'s shuttle row and the `claudeengine` open-launch paragraph say nothing about permissions today, so they only gain a sentence.
Scope's doc list leaves out the `buildOpenCmd` comment, which appears only under the optional Notes section, a hint and not a completeness requirement.
AC6's "no doc still says the open line passes no permission flag" could be read as covering code comments or not.
`Claude.OpenLaunch`'s doc comment lists what it does "before anything else" (effort, model), and D3 adds the mode check to that list.
That comment is also missing from Scope.

### [NIT] AC4 names an unobservable ordering

**Location:** `_lyx/discussion/decision-record.md`, AC4.
**Issue:** Minting a session id is an in-memory `newSessionID()` call inside `Claude.OpenLaunch` with no side effect.
No test at the public surface can observe whether it ran before the refusal.
The spec says a scenario names a behavior to cover.
The observable part of the ordering is that no strand was added and no marker directory was created, and AC4 already names both.

## Rejected

- D3's premise holds.
  `validatePermissionMode(mode, true)` returns no skip for `""` and `prompt` and a skip for `bypass`.
  Any other value returns the error naming the legal values.
  `Runner.Open` calls `OpenLaunch` before `newRunID`, `os.MkdirAll` of the marker directory and `reed.AddStrand`, so AC4's ordering can be met.
  The refusals that come before `OpenLaunch` (told error, empty role, unknown engine, covering pause, no `SessionOpener`) keep reporting first, as D3 says.
- D4's premise holds.
  `OpenLaunch` builds the fresh and resume lines from one `buildOpenCmd` call each, and `ResumeCmd` is `sh.OrElse(resumeLine, fresh)`, which is `a || b` in both shells.
  The line has no `WithEnv` wrapping, so a flag added in `buildOpenCmd` reaches both branches.
- D2 against `PATTERN-shuttle-provider-seam`: naming the mode words `bypass` and `prompt` outside the engine does not break the seam.
  `Spec.PermissionMode` is already documented as caller-owned vocabulary (`shuttleengine/doc.go`, `spec.go`), and `orchengine` already names `bypass`.
  The flag mapping stays in `claudeengine`.
  The honour-or-refuse sentence covers every `SessionOpener` engine, and `Runner.Open` already refuses an engine without the capability.
  No engine path is left undecided.
- Relocation: the deliberation in the decision record is limited to one-line rationales, which the spec allows under Decisions.
  The support log holds no requirement missing from the record: Q1–Q5 and A1 match D1–D3, and the rejected alternatives carry no constraint the plan writer needs.
- R1 holds: no plugin, contract or board stencil in this checkout points the orch at `lyx reed add` for a bypass session, so no extra Scope item follows from it.
