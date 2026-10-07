---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/09-verify-timeout-findings.md:1
    summary: card 9 makes a timed-out webster gate fail at once with no rerun, but contracts/specs/webster-spec.md and docs/overview.md still say a failing run is rerun once and no card edits either
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/10-verify-wait-note.md:1
    summary: card 10 shows the verify site in the note whenever it differs from the producer name, which is every gate site, so the note never reads verify running as decision D1 and its acceptance criterion require
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/16-batten-respawn-scoped.md:1
    summary: card 16 changes the spawn confirmation from machine-scoped to batten-process-scoped, but docs/overview.md still describes it as confirmed on this machine and neither it nor battenshed doc.go is edited by card 16
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/05-reed-resume-done-when.md:1
    summary: card 5 adds a not-hidden condition to the drop rule that decision D5 does not state, so a finished hidden strand lingers in state
---

### [MEDIUM] Card 9's no-rerun-on-timeout rule leaves the webster contract and the overview stating the old rule

**Location:** `_lyx/plan/09-verify-timeout-findings.md`, Edit list; `contracts/specs/webster-spec.md` section "The verify gate"; `docs/overview.md`, the webster paragraph.
**Issue:** Card 9 specifies that webster's gate does not rerun a timed-out first run and fails it at once.
That is a judgment beyond decision D3, which only says a timeout "rides the normal failed-gate path", and it makes two sentences untrue.
`webster-spec.md` says "A failing run is rerun once; a pass on the rerun passes the gate and records the failing identities as flaky".
`docs/overview.md` says "a failure is rerun once (a pass on the rerun records the identities as flaky)".
The plan edits `websterengine/doc.go` and `landingshed/doc.go` but neither of these two, though `webster-spec.md` is the consumer-facing contract for exactly this behavior and CLAUDE.md puts a changed observable behavior's docs in the same commit.
**Fix:** Add `contracts/specs/webster-spec.md` and `docs/overview.md` to card 9's Edit list.
State in the Intent that the spec's verify-gate section and the overview's webster sentence gain the exception: a run that exceeds the verify timeout is not rerun, since a hang is not flakiness, and it fails the gate at once with the timeout, log path and log tail as findings.

### [MEDIUM] Card 10's site rule contradicts the decision's status note

**Location:** `_lyx/plan/10-verify-wait-note.md`, third bullet of the Intent; `_lyx/plan/00-overview.md` Shared Decisions, first bullet.
**Issue:** Decision D1 gives the form `<producer>: <what> running <elapsed> (<detail>)` with the example `Webster-Burler: verify running 6m (go test -tags integration ./...)`.
Its acceptance criterion requires `lyx loom status` at each site to show producer, `verify running <elapsed>` and the command.
Card 10 renders `<producer>: verify[ <site>] running ...` and shows the site "only when it differs from the producer name".
The site labels `verifytree.Verify` receives are `webster gate`, `Webster-Burler gate`, `Publish` and `Finalize`, against producers `Webster`, `Webster-Burler`, `Publish` and `Finalize`.
So the two gate sites always differ and render `Webster-Burler: verify Webster-Burler gate running 6m (...)`, which is not the decision's example and does not contain `verify running`.
The producer already names the site at every site a run can wait at, so the clause adds nothing the decision asks for.
**Fix:** Drop the site clause from the rendered note: `<producer>: verify running <elapsed> (attempt <n>; <command>)`, the attempt clause still dropped when the attempt is 0.
`Marker.Site` stays in the marker.
Rewrite the `reviewstatus_test.go` sentence to drop "site shown and hidden", and reword the first Shared Decisions bullet so the status note's what is `verify` and the on-screen label is `verify <producer>`.

### [MEDIUM] Card 16's process-scoped spawn confirmation is documented nowhere

**Location:** `_lyx/plan/16-batten-respawn-scoped.md`, Edit list; `docs/overview.md`, the batten paragraph.
**Issue:** The overview says `Run-Shed` spawns the child's bootstrap "again while that status is `running` with no spawn confirmed on this machine", and the once-marker sentence just above it lists "a running child with no confirmed spawn".
Card 16 makes the confirmation hold only for the batten process that wrote it, so after a batten restart the running child is re-spawned, which "on this machine" no longer describes.
Card 17 rewrites only the halted-child sentence of that paragraph, and `battenshed/doc.go`, which card 17 edits for the halted wait, records no re-spawn behavior at all.
Decision D8 requires each touched package's `doc.go` to record its new behavior.
**Fix:** Add `docs/overview.md` and `internal/battenshed/doc.go` to card 16's Edit list.
Rewrite the overview's two spawn-confirmation clauses to say a running child is spawned again when the spawn was not confirmed by this batten process, and has `doc.go` record the same rule with its adoption of a child that has a live or retiring driver strand or a held run lock.

### [LOW] Card 5's drop rule adds a not-hidden condition

**Location:** `_lyx/plan/05-reed-resume-done-when.md`, third Intent bullet and the `planResumeLaunches` table sentence.
**Issue:** Decision D5 drops "a non-live strand whose done-when list is non-empty and whose every path exists", and its bound adds only that a live strand is untouched.
Card 5 drops only when the strand is "not live, not hidden", and pins a hidden strand as "neither launched nor dropped".
A hidden strand has no pane, so it is non-live, and shuttle accepts a hidden display.
A hidden ungated run whose outputs all exist would then linger in state for ever, which is the lingering D5's rationale says dropping avoids, and the first surface of such a strand would relaunch the finished agent, which is defect #409.
**Fix:** Drop the not-hidden condition from the drop rule: a non-live strand with a non-empty list and every path present is dropped, hidden or not, while the relaunch set stays non-live and non-hidden.
Change the hidden row to a hidden strand with a fully present list dropped, and add that a hidden strand with no list or a partly missing list is neither launched nor dropped.

## Focus questions settled

1. Verify coverage of the `tmux` tag: not a defect.
   The packages a card targets that hold `tmux`-tagged tests are `reedengine`, `reedcli`, `loomcli` and `webstercli`, and the `go test -tags tmux` line names all four.
   The other packages with `tmux` files (`pairteardown`, `standalonegeom`, `testkit/tmuxkit`) are targeted by no card.
2. Docs for changed behavior: the reed skip rule (card 3), the status `waiting` field (cards 10 and 12) and landing's verify (cards 9 and 13) are covered by their package docs, `docs/overview.md` and the status spec.
   Webster's verify gate and batten's spawn confirmation are not, per F1 and F3.
3. Card 5's tests: the `TestStrandOps_RealTmux` step reaches `Resume`'s state removal and the `Dropped` count, and the smoke check pins that the envelope carries `dropped`; no CLI path can set a done-when list, so a non-zero envelope value is unreachable and not a gap.
   The drop rule carries one condition the record does not state, per F4.
4. Card 16's Intent: not a defect.
   It routes a live or retiring driver strand, or a held run lock, to adoption and every other value to the spawn, states why retiring counts as live (D6), and bounds the re-spawn by "never stacks a second driver" and the hard error on a seam read failure.
