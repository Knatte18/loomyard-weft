# Decision record — decision-add-batch

## Goal

`lyx loom decision add` records several decisions in one call, lands them in one commit and sends each live agent one notice, so a batch of related calls no longer costs one commit and one full-text notice per decision.

## Scope

- `lyx loom decision add` gains `--from <file>`: a JSON array of entries, each with `title`, `decision`, `rationale` and the optional `pattern_entry`, `pattern_action` and `pattern_text`.
- `internal/discussionparser` gains a batch append that writes every entry in one write and runs the discussion check once.
- The notice of a batch names the new headings and points at the record instead of inlining every text.
- The verb's help, its file doc, `docs/overview.md` and the orch stencil's line on the verb name the batch form.

## Decisions

### Batch input is one JSON file named by `--from`

Decision: `--from <file>` reads a JSON array of objects with the keys `title`, `decision`, `rationale`, `pattern_entry`, `pattern_action` and `pattern_text`; `--by` and `--amends-plan` stay call-level flags and apply to every entry.
`--from` is refused together with `--title`, `--decision`, `--rationale` or any `--pattern-*` flag, and an empty array, an entry with an empty field, two entries with one title and two entries naming one PATTERN entry ID are refused before anything is written.

Rationale: a file carries multi-line texts and many entries without shell quoting, and JSON is what the verb's callers already write for `lyx board`.
Refusing duplicate titles and IDs keeps one heading and one `## PATTERN changes` bullet per entry, which the post-commit check and the bullet's replace-in-place rule both key on.

### One write, one check, one commit

Decision: `discussionparser.AppendDecisions` applies every entry to the record in memory, in input order, writes once and runs `Validate` once, restoring the prior bytes on a finding or an error; `AppendDecision` is the one-entry case of it.
The verb then commits once, and on a darn run checks that the committed record holds every entry of the batch.

Rationale: an all-or-nothing batch leaves no partial record for the caller to repair, and one commit is what the board entry asks for.
The verb's checks bound the batch exactly as they bound one entry: the same caller guard, the same darn-record checks, append-only labelled entries in one committed diff.

### `--amends-plan` with a batch appends every title

Decision: with `--amends-plan` every entry's title is appended to the plan amendment record, and the existing restore and commit rules cover the record's bytes before and after the whole batch.

Rationale: each entry is a decision that amends the plan, and the restore logic already works on whole-file bytes, so it needs no per-entry rollback.

### One notice per live agent; a batch's notice is short

Decision: each live receiver gets one notice per call.
A one-entry call keeps today's notice with the decision and rationale inline.
A batch's notice opens with the same fixed phrase, names every new heading and the record, and tells the agent to re-read the record's `Added after Discussion` entries, inlining no texts.
Webster's Merriam strand, which reads no decision record, gets the batch's texts inline in its one notice.

Rationale: the short notice removes the noise the board entry names, and Merriam keeps the only route by which a decision reaches it.

## Constraints

- `PATTERN-refusal-way-forward`: every new refusal names its way forward, and a test reaches it.
- `PATTERN-tolerant-records` does not apply: the batch file is caller input, read once; unknown keys are refused so a misspelt key is not silently dropped.
- `PATTERN-comment-line-breaks`: one sentence per comment line.

## Auto-mode assumptions

- The flag is named `--from`, the first form the board entry names; repeated `--pattern-entry`/`--pattern-text` pairs are not added.
- The success envelope keeps `heading` and `pattern_change` for a one-entry call and adds `headings` and `pattern_changes` lists for every call.

## Open risks

- A Merriam strand that reed names differently from `websterengine.MerriamStrandRole` would get the short notice and lose the batch's texts.

## Acceptance criteria

- A `--from` call with several entries appends all of them under `Added after Discussion` headings, in input order, in one write, and the record is committed once.
- A batch entry with a PATTERN change writes its bullet beside its entry.
- A batch whose result fails the discussion check leaves the record byte-identical to before the call.
- `--from` with a per-entry flag, an unreadable or malformed file, an unknown key, an empty array, an entry with an empty field, a repeated title and a repeated PATTERN entry ID are each refused with a way forward and nothing appended.
- A batch sends each live receiver one notice: a short one naming every heading and the record, and Merriam's carrying every entry's text.
- A one-entry call's behaviour and notice are unchanged.
- `--amends-plan` with a batch appends every title to the amendment record in the one commit.
