---
round: 6
focus:
  - "Receiving set: the notifier is a seam on `shuttleengine.Runner`, so the set of runners that notify is the set of construction sites that wire it. Check that `decision-record.md` D5 states that set once, by constructing package (`internal/loomcli/wiring.go` builds one shared runner; `internal/webstercli/wiring.go` and `internal/burlercli/wiring.go` each build their own; `internal/orchcli/cli.go` builds one for every orch verb; `lyx shuttle run` and detached runners build their own), that every site is either in the notifying set or in the wire-none list, and that the Auto-mode assumptions section refers to the same set in the same wording. Settle whether an autonomous strand spawned through `lyx webster recover-batch` (`internal/webstercli/recoverbatch.go`) has a stated receiver or a stated bound, and whether D5 names any site that builds no runner (`internal/battencli`, `internal/battenshed`)."
  - "Cadence: check that the Scope bullet, the D5 title, the D5 body and the Acceptance criteria state the same notification cadence for a held turn end, including the Attach case (D5 body: an Attach notifies at most the last held turn end past the notified offset), and that each stated cadence rule has an acceptance criterion that pins it."
---

Both entries are derived from the round 5 findings still open; neither caps severity or presumes an outcome.
