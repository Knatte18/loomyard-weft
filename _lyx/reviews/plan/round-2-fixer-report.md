# Round 2 fixer-report — plan

Every finding's premise checked out against the code, and each was fixed as proposed.

## Changes

- **F1** (`_lyx/plan/06-webster-wiring.md`): added `internal/websterengine/doc.go` to **Edit:** after `internal/webstercli#configLoadError`.
  The refusal-spec paragraph now also extends the "config file unreadable" row's trigger with `language.yaml`, keeping its class and way forward.
  It also has the doc.go "A broken config stops Master" section name `language.yaml`: unusable content refuses with `config_invalid: true` and stops Master, and an unreadable file refuses transient.
- **F2** (`_lyx/plan/01-language-config-module.md`, `_lyx/plan/05-loom-wiring.md`): card 1's unreadable-file bullet now says loom's `wire` returns that error as the plain arming error, with no `lyx config language` clause.
  The old wording said loom's existing transient handling classifies it, and `shedtransient.Mark` runs only in `loomPreStep`, after arming.
  Card 5 now says an unreadable `language.yaml` surfaces through the shed section's `step arming refused` row.
- **F3** (`_lyx/plan/05-loom-wiring.md`): the new row now goes in the shed section, after "hub config missing" (`contracts/specs/refusal-spec.md:174`).
- **F4** (`_lyx/plan/05-loom-wiring.md`, `_lyx/plan/06-webster-wiring.md`): both row triggers now include a worktree local override file that holds `skills`, which the Guard classification refuses.
- **F5** (`_lyx/plan/01-language-config-module.md`): `Load`'s error now ends with `; way forward: fix it with "lyx config language" from the prime`, the same shape as `batcher`'s `retiredKeyError`.
  The row cells in cards 5 and 6 still quote only the clause content.
- **F6** (`_lyx/plan/08-burler-profile-key.md`): `burlerRoundProfile`'s doc comment now has to state eight recognised keys, `language-skills` among them, and that seven of the eight names duplicate `profileYAML`.

`echo '{}' | lyx loom validate-plan --overlay` returns `ok: true`, with informational findings only.

## Disputed

None.

## Deferred

None.
