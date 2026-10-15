---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md`, D2 arms follow and moves the viewer when `lyx loom start` runs. Settle whether D2's Bound names every kind of entry that re-arms follow: a fresh run, a re-entrant start on a live run, and a resume of a parked run by the orch, batten or an approve/reject follow-up, since `lyx loom start` is also the resume verb (see `internal/loomcli/approve.go`, `internal/loomcli/reject.go`, `internal/battencli/wire.go`). Settle also whether the Acceptance criteria pin the chosen resume behavior with follow off."
  - "In `_lyx/discussion/decision-record.md`, the Acceptance criteria must contain a row that fails if D4 (attach selects the relevant window while another client is attached) is left unimplemented. Settle whether some row attaches a second client while an overview window already exists, with follow on and with follow off."
  - "In `_lyx/discussion/decision-record.md`, settle whether each Acceptance criteria row agrees with the Decisions D3, D4, D6 and D7, in particular any row on replacing an overview strand against D6's keeping of the viewer move in `ReplaceStrand` (`internal/reedengine/replace.go`), and whether D3's and D4's stated bounds name every disturbance they allow, including `lyx shuttle run --focus` (`internal/shuttlecli/run.go`) as a `Focus` user and a viewer moved by tmux keys outside reed's bindings."
  - "In `_lyx/discussion/decision-record.md`, settle whether D7 decides what the follow button's click from off does when the lyx path, socket path or tmux path cannot be resolved at pin time, the case where `switchBindingCommandsLocked` in reedengine leaves the session-switch keys unbound."
---

Round 2 targets the decision record's bounds and acceptance rows.
