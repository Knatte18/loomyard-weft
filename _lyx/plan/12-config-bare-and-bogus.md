# Card 12 — bare lyx config lists, an unknown argument refuses

**Edit:**
- `internal/configcli#Command`
- `internal/configcli#dispatch`
- `internal/configcli#runConfig`
- `internal/configcli#buildConfigLong`
- `internal/configcli/configcli_test.go`
- `internal/configcli/doc.go`
- `docs/overview.md`

**Delete:**
- `internal/configcli/menu.go`

**Intent:**
Implement D9.
`Command`'s `RunE` decides before `runConfig` resolves any cwd:

- No argument and neither `--print` nor `--set`: print the command's help, which names `reconcile` and every `configreg` module, exit 0.
- An argument that is not a `configreg` module: refuse with an error envelope, exit 1, whose message contains `unknown subcommand`, names the argument and says `run "lyx config" to list modules and verbs`.

`lyx config <module>` and bare `lyx config` with `--print` or `--set` keep their present paths, which resolve a cwd.
A subcommand name always routes as a subcommand.
The interactive picker goes: `menu.go` is deleted and `dispatch`'s no-module, no-flag branch with it.
`buildConfigLong` stops describing the menu and says bare `lyx config` lists modules and verbs.
Grep for the menu's wording and for `menu(`; the `TestMenu*` tests go with it.

Bound: the new refusal fires only for an argument that is neither a subcommand nor a module, which today already fails; nothing that routed before is refused.

Tests, untagged: bare `lyx config` from a non-git temp dir exits 0, prints no envelope and names `reconcile` and every module; `lyx config bogus` there exits 1 with an envelope containing `unknown subcommand` and `bogus`; no `configreg` module shares a name with a `config` subcommand.

`doc.go` and `docs/overview.md`'s `lyx config` line describe the listing and drop the picker.

**ImpactSummary:** `lyx config` with no argument or an unknown one; module, `--print`, `--set` and `reconcile` paths are unchanged.

**Commit:** 12: bare lyx config lists, an unknown argument refuses
