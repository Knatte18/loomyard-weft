# orch-rules-files: hub-wide and own rules files replace ORCHESTRATOR.md

Batch 01 (rule-files-render) finished first time, head `0ba9e9557c7b5a793e203f0858b432c7f81e96d2`, with no deviations and no departures.
The orch role file now inlines a hub-wide rules file and the orch thread's own rules file, through told paths on `orchengine.Paths`.

Batch 02 (orch-rules-verb) held cards 2, 3 and 4.
Its first `record-batch` failed the done-check, because card 4's create target `.scratch/orch-rules-move.md` no longer existed after the fork filed the `orch-rules-move` board note and removed the scratch file.
One backgrounded `recover-batch 02` ended `done` at head `8f0a8633d7b3c3fe6ca496be04937830529e88cf`, with one recovery.
It built `lyx orch rules --from <file>`, deleted `ORCHESTRATOR.md` and reworded `CLAUDE.md` and the role stencil's placement sentences.
The board note is task 662.

The scope guard reported one deviation, `internal/orchcli/rules.go`, which is informational.
No batch reported a `Pattern-Departure:` trailer.
