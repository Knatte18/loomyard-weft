---
round: 2
focus:
  - "internal/battencli/cleanup.go `clearMotifRun` and `removeRunRecords`: when the records commit fails after the run directory was deleted from disk, can `lym beam create` (`createSteps` in internal/beamcli/create.go) run again and finish the removal, or does it start a new pawl run over a tracked-but-deleted status, seed and child-recipe record?"
  - "internal/battencli/wire.go Seed-Child `WriteSeed` seam, where the child-recipe record is written through `shedrun.WriteChildRecipe`: does a test in internal/battencli/wire_test.go run the seam for the motif recipe and fail when the write is removed, reordered before the child seed, or records a value other than the child's seed recipe?"
  - "internal/battencli/cleanup_test.go `TestClearMotifRun_RemovesOnlyADoneMotifRun`: does the table cover a records commit failure, a scratch-empty failure, a push-spawn failure and a done motif run the records HEAD no longer tracks, each against the `cleared` result and the error the code returns?"
  - "docs/overview.md beam entry and the `lym batten cleanup` paragraph: do they name that `lym beam create` clears a done motif run, as internal/beamcli/doc.go and internal/battencli/cli.go in the same diff do?"
  - "Doc comments on `clearMotifRun` in internal/battencli/cleanup.go and `ReadChildRecipe` in internal/shedrun/childrecipe.go: does each read on its own terms, with no dependence on another symbol and no history about runs seeded before the record existed?"
---

Focus entries name where to look; the review's own judgment sets severity.
