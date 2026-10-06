# Card 17 — stencil-misc-prune

**Edit:**
- `internal/stencil#`
- `internal/stencilcli#`
- `internal/stencilstore#`
- `internal/state#`
- `internal/agentname#`
- `internal/batcher#`
- `internal/buildinfo#`
- `internal/clihelp#`
- `internal/cliwire#`
- `internal/friction#`
- `internal/frictionengine#`
- `internal/idecli#`
- `internal/ideengine#`
- `internal/modelspec#`
- `internal/pattern#`
- `internal/quarrycli#`
- `internal/selfreportcli#`
- `internal/selfreportengine#`
- `internal/shell#`
- `internal/tokenvocab#`
- `internal/vscode#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on the test files of the remaining in-scope packages: the stencil packages, `internal/state` and the small CLI and leaf packages listed above.

`internal/stencil` has a large candidate set with many `Removable: no` entries, which are folded or kept, never deleted (Decision 2); a check on rendered stencil text is a unique assertion unless its covering test pins the same text.
`internal/selfreportcli`'s `TestGroup_BareListsCreate` is a `failed` entry (Decision 8).
`internal/cliwire`'s `specsseed_integration_test.go`, `internal/stencilcli`'s `cli_integration_test.go`, `internal/idecli`'s `cli_test.go` and `internal/ideengine`'s `integration` files hold this group's tier-move and scenario-merge leads.
`internal/agentname` tests keep one row per name form (`PATTERN-agent-name`).

**ImpactSummary:** Test files of the remaining small in-scope packages only; no production change.

**Commit:** 17: Prune stencil, state and leaf package tests
