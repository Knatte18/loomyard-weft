# Card 1 — decision-note-title

**Create:**
- `internal/orchengine#DecisionNoteTitle`

**Edit:**
- `internal/orchengine#RenderRoleFile`
- `internal/orchengine#orchengineAllowedImports`
- `internal/loomcli#decisionWayNoRecord`

**Intent:** D1 keeps the decision-note title form `Decision for <slug>:` once `internal/alignment` goes, so move its one declaration into `internal/orchengine`, the package that renders it into the orch role and that `internal/loomcli` may import.
Declare `DecisionNoteTitle(slug)` in `internal/orchengine/prompt.go` with the doc comment the alignment copy carries (the colon keeps a slug from matching a longer one, the match is case-sensitive), naming its callers as the orch role render and `lyx loom decision add`'s no-record way forward, and no snapshot filter.
`RenderRoleFile` fills `decision_note_title` from it, and `decisionWayNoRecord` builds its text from `orchengine.DecisionNoteTitle("<slug>")`.
Drop `internal/alignment` from `orchengineAllowedImports`, since no production file of the package imports it any more, and the stale-entry check fails otherwise.
`alignment.DecisionNoteTitle` stays until card 5 deletes the package; its only remaining caller is `internal/loomcli/alignmentwiring.go`, which card 3 deletes.
This is a pure move: `internal/orchengine`'s prompt test and `internal/loomcli`'s decision test already assert the rendered `Decision for <slug>:` text.

**ImpactSummary:** Two callers move to the new declaration; no behavior or text changes.
