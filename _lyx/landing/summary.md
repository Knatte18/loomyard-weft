# lyx loom decision add: record several decisions in one call

`lyx loom decision add` took one decision per call, so a batch of related decisions cost one commit and one full-text creel notice each.
Shortening 20 PATTERN entries took 20 calls, 20 commits and 20 notices into the plan session, each repeating the whole decision text.

The verb now takes `--from <file>`, a JSON array of entries.
Each entry has `title`, `decision`, `rationale` and the optional `pattern_entry`, `pattern_action` and `pattern_text`.
`--by` and `--amends-plan` stay call-level flags and apply to every entry.
All entries land in one write and one commit, or none does, and each live agent gets one notice for the call.

## Behaviour

- A batch appends every entry under its own `Added after Discussion` heading, in input order.
  An entry with a PATTERN change gets its bullet in `## PATTERN changes` beside it.
- The batch notice opens with the same fixed phrase as today's, names every new heading and the record, and tells the agent to re-read the record's `Added after Discussion` entries.
  It inlines no texts.
- Webster's Merriam strand reads no decision record, so its one notice carries every entry's decision and rationale.
- With `--amends-plan`, every entry's title is appended to the plan amendment record, in the same commit.
- The success envelope gains `headings` and, when any entry has a PATTERN change, `pattern_changes`.
  A one-entry call keeps `heading` and `pattern_change` as before.
- A one-entry call behaves as before, and its notice is unchanged.
- `--from` is refused beside `--title`, `--decision`, `--rationale` or any `--pattern-*` flag.
  It is refused for an unreadable file, a file that is not one JSON array of objects, data after the array, an empty array, an unknown key, an entry with an empty field or a partial PATTERN-change form, two entries sharing a title, and two entries naming one PATTERN entry ID.
  Each refusal names its way forward, and nothing is written.

## Design decisions

- `discussionparser.AppendDecisions` applies every entry in memory, writes once, runs the discussion check once, and restores the prior bytes on a finding or an error.
  `AppendDecision` is its one-entry case, so the all-or-nothing rule needs no per-entry rollback.
- The verb's guards bound a batch as they bound one entry: the caller guard, the darn-record checks, and append-only labelled entries in one committed diff.
  After the commit, the committed record must hold every entry of the batch, otherwise the call is refused.
- Duplicate titles and PATTERN entry IDs are refused up front because the post-commit check keys on headings, and the PATTERN bullet's replace-in-place rule keys on the ID.
- Unknown keys are refused rather than ignored, so a misspelt key does not drop a field silently.
  That is a separate pass over the file, because the entries decode itself stays tolerant.
- The flag is `--from`; repeated `--pattern-entry`/`--pattern-text` pairs are not added.
- Merriam is told apart by its strand role, `websterengine.MerriamStrandRole`.
  A Merriam strand named differently would get the short notice and lose the batch's texts.

## Docs

The verb's help and file doc, `internal/discussionparser/doc.go`, `docs/overview.md` and the orch stencil's line on the verb name the batch form.
The verb's long help was condensed while the batch form was added; its rules for the single-entry form are kept.

## To check by hand

- Run a real `lyx loom decision add <slug> --by parent --from decisions.json` against a live run with a Merriam strand, and confirm that Merriam gets the inline notice and the other agents the short one.
  The tests cover the notice text and the routing by role, not a live tmux run.
