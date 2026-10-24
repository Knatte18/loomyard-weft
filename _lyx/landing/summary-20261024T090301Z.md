# lyx loom decision add records a batch of decisions in one call

`lyx loom decision add` took one decision per call, so a parent recording many related decisions, such as twenty PATTERN entry rewrites, paid one commit and one full-text creel notice per decision.
The verb now takes `--from <file>`, a JSON array of entries, and records the whole batch in one write, one commit and one notice per live agent.

## What changes

- `--from <file>` replaces the per-entry flags with a JSON array of objects with the keys `title`, `decision`, `rationale` and, for a PATTERN change, `pattern_entry`, `pattern_action` and `pattern_text`.
  `--by` and `--amends-plan` stay call-level and apply to every entry.
- `discussionparser.AppendDecisions` applies the entries in order in memory, writes the record once and runs the discussion check once, restoring the prior bytes on a finding or an error.
  `AppendDecision` is now its one-entry case.
- The verb commits once.
  On a darn run, the post-commit check refuses when the committed record lacks any entry of the batch, and names the first one missing.
  With `--amends-plan`, every title is appended to the plan amendment record, under the existing whole-file restore rules.
- Each live receiver gets one notice per call.
  A one-entry call's notice is unchanged.
  A batch's notice names every new heading and the record, and tells the agent to re-read the record's `Added after Discussion` entries.
  Webster's Merriam strand reads no decision record, so its one notice inlines every entry's decision and rationale.
- The success envelope adds `headings` and, when any entry carries a PATTERN change, `pattern_changes`.
  `heading` and `pattern_change` stay for a one-entry call.
- The verb's help, its file doc, the `discussionparser` package doc, `docs/overview.md` and the orch stencil's line on the verb name the batch form.

## Refusals

`--from` is refused, with a way forward, when it is combined with any per-entry flag, or when the file is unreadable, is not one JSON array of objects, has data after the array, is empty or carries an unknown key.
It is also refused when an entry has an empty field or a partial PATTERN-change form, or when two entries share a title or a PATTERN entry ID.
Every refusal comes before anything is written.

## Design decisions

- An unknown key is refused by a separate key pass over the file (`unknownEntryKey`), not by `DisallowUnknownFields`, because `PATTERN-tolerant-records` and its scan forbid that option in production code.
  The entries decode itself stays tolerant.
- Duplicate titles and PATTERN entry IDs within a batch are refused rather than resolved, because the post-commit check keys on headings and the `## PATTERN changes` section holds one bullet per ID.
- Merriam is told apart by its strand role, `websterengine.MerriamStrandRole`; a strand named otherwise would get the short notice and lose the batch's texts.
- The flag is `--from` only; repeated `--pattern-entry`/`--pattern-text` pairs are not added.

## Tests

- `TestAppendDecisions_WritesBatchWholeOrNothing` covers a batch landing in order with its PATTERN bullet, and an empty batch, a later empty field and a later finding leaving the record unchanged.
- `TestDecisionVerb_FromFileRecordsBatch` covers one append and one commit, the envelope's lists, both titles in the amendment record, and the short and inline notices.
- `TestDecisionVerb_Refusals` gains a row for each `--from` refusal and for a darn batch whose commit lost a later entry.

## Check by hand

- `internal/loomcli/decision.go`, `decisionEntries`: an unknown or misspelt key in a `--from` file is refused; the key pass was added late and is worth one real call.
- `internal/loomcli/decision.go`, `decisionEntries`: a `--from` file with a stray `]` or `}` after the array is refused as data after the array.
- `internal/loomcli/decision.go`, `beginAmendment`: a `--from` batch with `--amends-plan` whose second title fails to append restores the amendment record.
- `internal/discussionparser/doc.go`: the package doc names `AppendDecisions`.
- `internal/loomcli/decision.go`, the `add` help: it says `--title`, `--decision` and `--rationale` are required unless `--from` is passed.
- `internal/discussionparser/append.go`, `AppendDecisions`: the doc comment states the behaviour directly.
