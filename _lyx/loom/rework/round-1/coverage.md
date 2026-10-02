# Rework coverage — PR #335

- **F1: rename the repo's short code from "code" to "shortname"** — cards 17, 18, 19, 20.
  Card 17 renames agentname's grammar symbols (`ValidateShortname`, `StandaloneShortname`, `Name.Shortname`) and its texts.
  Card 18 renames the record to `.lyx-shortname`, the flag to `--shortname`, the verb to `lyx fabric shortname [<shortname>]`, the envelope keys, the fabric symbols and files, `hubforge.TestShortname`, the sandbox tool, every refusal and warning naming them, and the refusal-spec rows.
  Card 19 renames reed's told `NameCode` to `NameShortname`, its tellers, reed's help texts and the reed-side test names.
  Card 20 updates `CONSTRAINTS.md`, `docs/overview.md` and the fabricengine, reedengine and hubgeom package docs, and its Verify greps the repo for any "code" spelling of the concept left behind.
