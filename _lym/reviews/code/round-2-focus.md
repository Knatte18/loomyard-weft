---
round: 2
focus:
  - "internal/boardengine/replay.go, the replayFields table: does a board sync replay carry Task.RecordBodySHA256 together with the record body, as the decision record's stamp-travels-with-the-entry rule and boardengine/doc.go state?"
  - "internal/boardengine/store.go, the merge path's keepDiscussed branch: when an entry is removed and re-upserted, does the entry keep both the discussed flag and RecordBodySHA256, so promote still warns for a changed Sketch body?"
  - "internal/boardengine/recordupsert.go and recordupsert_test.go, the family-project exemption (Kind project, held, filed project equal to the slug): do the tests show it never exempts a non-filed project, a filed part of another project, or a run-held family project?"
  - "internal/boardengine/recordupsert.go and internal/boardcli/cli.go, the probe-failure refusal: is its way forward correct for a nil location under --board-path and for a config-load failure, and does a test reach that path?"
  - "internal/boardcli/cli.go and recordfile.go: does the CLI probe's skip behavior match what Add's real callers pass, and do the flag help and doc.go state whether --record-file reads stdin?"
  - "plugins/ly/skills/board/SKILL.md: does it document upsert --record-file, its refusals, and promote's warning for a Sketch body edited after the record was stored?"
---

Each entry restates its own sites and question; the reviewer settles them against the diff.
