---
round: 1
focus:
  - "internal/loomshed/darnwrite.go `renderPriorWork`: does it tell prior work whenever a Darn-Write history entry exists and tell none otherwise, as the acceptance criteria require? Check the `latest.Outcome` wording against the `shedengine.Outcome` values a history entry can hold."
  - "internal/loomcli/darnwiring.go `latestDarnOutcome`: the halt-reason read is gone and `FindingsPath` is set only when the file exists. Settle whether the `os.Stat` error handling (not-exist versus other errors) is right, and whether `shedrun.GateFindingsFile(l, runID, loomshed.NameDarnWrite)` is the file Env.GateFindingsFile gives the row."
  - "internal/shedrecipe/entries_darnwrite.go: the loop sets `Reprompt` on `gate[i]` after `resolveRowGateSpec`. Settle whether that mutates shared or cached state, whether every gate entry of the built table (including tables from `env.DarnTable`) carries the line, and whether `env.DecisionRecordPath` can be empty or unset here."
  - "`DarnGateReprompt` wording: it must stay one line and name the findings path and the decision record. Settle whether 'bar one a decision ... forbids' correctly covers several forbidden findings, and whether it keeps the board entry's bound that only an `Added after Discussion` entry may excuse a finding."
  - "Scope against the decision record: the diff must not touch other rows' re-prompts, the shed engine's resume write, or add a resume refusal. Settle whether tests changed only as rows of the existing tables, and whether any removed `Reason` assertion left a gap."
  - "Docs: `internal/loomshed/doc.go` and the `entries_darnwrite.go` doc comment must match the new behavior. Settle whether any other doc, `docs/overview.md` or PATTERN text still describes the halt-reason note."
---

Seed focus from the board entry, the decision record and the one-commit diff.
No parent decisions beyond the record were read.
