# Test tiers: every hub test run goes through `lyx gate test`

## What changes

`lyx gate test` stops being a `go test` wrapper.
A hub declares numbered tiers in its hub-wide `gate.yaml`, and lyx hard-codes no test command.
A C# hub gets the same slotted, bounded, observable gate as this Go hub.

Each tier in the open map `tiers` (`tier0`, `tier1`, ...) holds `about`, `command` (placeholders `{targets}`, `{parallel}`, `{args}`), `dir`, `env`, `all_targets`, `parallel`, `timeout_min`, `counter` (`go-json` or `vstest`), `covers`, `slots` (a count or `all`) and `prebuilt_lyx`.
Top-level keys add `gates` (gate name to tier list), `scoper` (`go` or `none`), `guard_tier` and `refuse_raw`.

`lyx gate test [--tier N]... [--card <id>]... [--base <rev>] [--all] [--guards] [--wait] [<targets>...] [-- <args>]`:

- Tiers run ascending and the call stops at the first failing tier; a tier another listed tier `covers` is skipped.
- Scope is computed by lyx against the parent by default; `--base`, `--card`, `--all` and explicit targets are the other forms.
  Under the `go` scoper, `internal/impactset` yields targets per module, and the module's `//lyx:guard` tests outside the scope still run through `guard_tier`.
  Under `none`, a computed scope is whole.
- Each tier takes its own slots and releases them before the next tier.
  A tier asking for several slots, or `all`, takes a drain lock first and then the slots in ascending order, so single-slot callers cannot starve it.
- Each tier has a mandatory timeout; a run past it has its process tree killed and fails with a timeout line.
- Each tier run writes a JSON record and the full output log under the board's gate directory, pruned after 14 days, so a log outlives the worktree.
- A run that discovers zero tests passes, in every scope.
  The discovered count is still logged; there is no minimum-test floor.

Every verify site renders its gate's tier list as `--tier` flags with `--wait`: the Webster card, batch and final gates, the round gate, Merge-In, Finalize and Publish.
Card `**Verify:**` lines run through the new `lyx gate card-verify <card id>`, which the fork prompts render in place of the lines.
`verifytree.Verify` no longer takes a slot or applies its own 60-minute timeout, since every command it runs is a bounded gate call.

The shell guard refuses a hub's `refuse_raw` prefixes (such as `dotnet test`) with a steer to `lyx gate test`.
The plan gate refuses a `**Verify:**` line holding such a prefix, or a `lyx gate test` call without `--wait`.

`lyx config --set` writes a whole open-map entry as a YAML flow mapping or list, which is how the orch sets a tier.

## Retired

- `--tags` on `lyx gate test`, refused with a steer to `--tier`.
- `gate.yaml`'s `go_parallel`, replaced by per-tier `parallel`.
- `landing.yaml`'s `publish_verify`, replaced by the `publish` gate.
- `darn.yaml`'s `verify`; darn sites run their gate lists, and a non-empty `tree_scope_verify` must be one `lyx gate test --wait` call.
- `language.yaml`'s `verify_tiers`; prompts render the hub's tiers and gate lists instead.
- The plan's `## verify:` section as a gate input.
  The parser still accepts it, and the plan gate reports it as informational.

## Design decisions

- Tiers live in the hub-wide `gate.yaml`, so all worktrees of a hub share them.
- A tier never includes a lower one by definition; `covers` states it where the command runs a lower tier anyway (Go's tagged runs also run the untagged tests).
- Non-tier slot holders keep one slot and set their own `-trimpath` and `-p` flags: a card `**Verify:**` line that is not one gate call, the `internal/planglyph` type load, and `cmd/testtiming` in every mode.
  A gate call inside any of them inherits the slot.
- The `go` scoper narrows one step further than before: an empty set runs no tier command.
  `go build ./...` for such a diff is left to the whole runs at `final` and `landing`.
- `CLAUDE.md`'s push rule is now `lyx gate test --tier 0 --tier 1 --tier 2` on `main`.
  The PATTERN files and the module docs (`gatecli`, `gateslot`, `impactset`, `landingshed`, `loomshed`, `darnengine`, `languageconfig`) carry the new rules.

## To check or run by hand

- After the deploy, the orch must set both hubs' tiers with `lyx config gate` before the next gate run.
  Until then every gate in an in-flight run is refused naming `lyx config gate`.
  The loomyard values are in the `internal/gatecli` doc example; set each tier's `timeout_min` to 60 first, then fit it to the measured whole-run wall time.
- An in-flight plan's card `**Verify:**` line holding `--tags` fails its card re-run after the deploy; rewrite it to `--tier` with `--wait`, then run `lyx webster rebaseline` naming that card.
  A line that is a `lyx gate test` call lacking `--wait` takes a single slot and waits without the keep-cache-warm cap.
- Processes still running the pre-deploy binary, such as an in-flight card runner or `verifytree`, keep taking single slots without seeing the drain lock until they exit.
- The `vstest` counter's summary-line shape comes from the repo notes, not from a NORCE Models run; a dotnet version printing another shape logs wrong counts.
- The `go-json` render must keep failure lines, panics and build errors readable; check one failing Go tier by eye.
- On Windows a timeout kills only the tier's started process, not its tree.
- Task `tuned-defaults-template` changes how reconcile treats seeded template values; the order of the two landings decides whether seeded `gates` lists override the orch's values.
- Running task `origin-reconcile` edits `internal/landingshed` and `internal/loomcli`, so expect merge-in conflicts.
- Task `darn-verify-fixes` assumes `publish_verify`, now retired, and needs re-scoping after this lands.
- Review findings left open, to check by hand:
  - An absent `gate.yaml` is refused at wiring (`absent-gate-yaml-refused-at-wiring`): confirm that a hub with no worktree-layer `gate.yaml` gets the `lyx config gate` refusal where the gate is wired, and no other error.
  - `decodeconfig-resign-arrow-missing`: confirm the config decode path for the tier keys carries the arrow in its re-sign step as its siblings do.
  - `shared-fixtures-missed-by-new-behavior`: confirm shared test fixtures still match the new behavior; the final commits strip the retired `go_parallel` key from the remaining fixtures.
  - `triage-section-names-retired-verify-suite`: the integration-suite triage text may still name the retired verify suite; check its wording against the tier names.
- The verify run's one failure was `internal/configsync` `TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate/landing_absent_with_a_prime_copy_is_seeded_from_the_prime`.
  The seeded `landing.yaml` row still used the retired `publish_verify`.
  Commit `bf5edc9d` fixed it with a live key; check that the seeded row reads right.
