# Card 12 — darn-review-answer-key

**Edit:**
- `internal/loomcli/doc.go`
- `contracts/recipes/darn-recipe.yaml`
- `contracts/stencils/loom/loom-rubric-code-review.md`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#writeBoardEntrySnapshot`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWriteBoardEntrySnapshot`
- `internal/loomrecipe#TestDarnPerch_SharesLoomsPerchBarTheAnswerKey`
- `docs/overview.md`

**Delete:**
- `internal/loomcli#TestCommitDarnDiscussion_Real`
- `internal/loomcli#commitDarnDiscussion`

**Uses:**
- `internal/loomcli#commitDiscussion`

**Intent:** On every loom recipe the Code-Review perch measures against the decision record (D7); a darn run no longer writes the board-entry snapshot.

- `darn-recipe.yaml`'s Code-Bouncer `artifact_paths` and Code-Burler-Review `fasit.paths` name `_lym/discussion/decision-record.md` alone. The fasit instructions say the decision record is the task's scope: the board entry's body under `## Goal`, as amended by its `Added after Discussion` entries, which win where they differ. The header's answer-key paragraph says so, dropping the snapshot.
- `wireDarn` sets `env.CommitDiscussion` to `commitDiscussion(location)`, the plain discussion commit, and `commitDarnDiscussion` goes, so no darn commit writes `_lym/discussion/board-entry.md`.
- `writeBoardEntrySnapshot` stays for pawl, whose perch reads the snapshot until `pawl-loom-recipe` lands; its doc names the pawl run alone. `darnengine.BoardEntrySnapshot` stays for the same reason.
- The Code-Review rubric's answer-key forms become: a plan (the decision record and the plan), a decision record alone (darn: the record whose `## Goal` holds the board entry, as amended by its `Added after Discussion` entries), and a board entry snapshot (pawl). Every bullet the rubric scopes "with a board entry snapshot" (an unrecorded parent correction, a step that edits the task's own board entry, stencil wording) applies to the decision-record-alone form too, read against the amended entry in the record.
- A darn run in flight past the old Darn-Write is reviewed against the record its own writer wrote, with its `Added after Discussion` entries.

Tests: `TestDarnPerch_SharesLoomsPerchBarTheAnswerKey` pins the darn answer key as the decision record alone; `TestWire_DarnRecipe`'s discussion-commit subtest asserts the darn commit seam writes no `board-entry.md`; `TestWriteBoardEntrySnapshot` keeps the pawl snapshot. `docs/overview.md`'s darn answer-key line names the record.

**ImpactSummary:** Darn's perch answer key and commit seam change; pawl's snapshot path is untouched, and the rubric gains a third answer-key form.
