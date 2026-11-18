# Run tierless hubs and inert-only changes; route one-class test runs

Two bugs met in a new C# hub, both in the gate and the guidance around a hub test command.

## A hub that declares no tier

The gate template declares `tiers: {}` while every gate key names tier 0,
so `gateslot.Config.Gate` refused every key in a fresh hub.
Shed build renders the Darn-Write fixer's gate call through it, so every run failed before its first producer, whatever its diff.

- `Config.Gate` (`internal/gateslot/tiers.go`) now resolves a key to no tiers, and no refusal, when the hub declares no tier at all.
  A hub that declares some tiers but no `tier0` is still refused, as is a missing key, an empty list or an undeclared tier.
- `lym gate test --gate <key>` (`internal/gatecli/run.go`) passes at once in such a hub, printing that the hub declares no tier and that `tier0` is declared with `lym config gate` once the repository has tests.
  A call without `--gate` there is still refused, since it asks for a tier by name.

## Finalize over an inert-only change

A gate key of scope `all` (Finalize, webster, parent-align, adopt) ran every tier whole, ignoring `inert_paths`.
A docs-only branch in a repository with no .NET project then failed its landing on `dotnet test .`.

- `allScope` (`internal/gatecli/scope.go`) gives a `--gate` key of scope `all` the empty scope when every file changed since the merge base with the parent branch matches `inert_paths`.
  Each tier then runs no command, as an inert-only computed scope already did; a `carry: false` tier still runs whole.
- An explicit `--all`, a worktree with no parent branch and a merge base that cannot be read stay whole.

## The silent `dotnet test`

lym's guard does not neutralise `dotnet test`: a hub whose `refuse_raw` lists it gets a deny that names the refused part and the way forward.
The silent exit 0 is a test project that sets `IsTestProject` false unless an MSBuild property is passed,
which the gate already reports with its "ran no test of" line.
The gap was that the gated darn writer is told to run no gate, so it had only a raw test command for measuring a value.

- The gated darn fragment (`contracts/stencils/darn/darn-fragment-make-change-gated.md`) now names one exception:
  to measure a value only a test run shows, run that one test class once through `lym gate test --tier <n> <target> -- <args>`.
  It says a raw run can exit 0 having run no test, and to name to the parent a project the gate reports as running none.

## Design decisions

- The no-tier case passes rather than flagging a missing `tier0` at hub setup: the issue's preferred fix,
  and it lets a hub with no tests run docs and code tasks alike until tests exist.
- The measuring route goes through `lym gate test` rather than naming `dotnet vstest`, so the stencil hard-codes no language (`PATTERN-code-conventions`), and the gate's no-test notice covers the run.
- The inert check applies only to scope `all` reached through a gate key, so a caller's explicit `--all` still runs everything.

## Tests

- `internal/gateslot/config_test.go`: `TierList` rows for a hub that declares no tier, with a held and a missing key.
- `internal/gatecli/test_integration_test.go`: a `--gate finalize` call in a hub with no tier exits 0, names the missing tiers and runs no tier command.
- `internal/gatecli/scopes_integration_test.go`: `--gate finalize` over a Markdown-only diff runs no tier command and names the inert diff.
