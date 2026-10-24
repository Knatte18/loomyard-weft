# Gate runs once: per-producer gate keys, tier-named tags, row entry gates

Each test now runs once per tree and in one tier.
Every producer that runs a test gate names its own `gate.yaml` key, `lyx gate test` carries a green tier forward over an unchanged tree, and shed rows get an entry gate.

## What changes

- **Gate keys.**
  `gate.yaml`'s `gates` maps each key to `{tiers: [...], scope: card|diff|all}`.
  Webster's keys are `card`, `batch`, `webster` (formerly `final`) and `webster-fix` (formerly `round`).
  Recipe rows read `merge-in`, `finalize`, `darn-write` and `code-burler-fix`, or the key their `gate_key` names; `landing` is retired.
  `lyx config reconcile` migrates a hub file, giving each key the scope closest to today's call.
  `guard_tier` is pruned.
- **`lyx gate test --gate <key>`.**
  It resolves the key's tiers and scope at run time and is refused beside `--tier`, `--upto`, `--all` or `--base`, and for a `card` scope without `--card`.
  Explicit targets replace the key's scope.
  The run record gains the gate key.
- **Scopes.**
  A `diff` scope takes its base from the newest ancestor commit the green record holds whole-green for that tier, else the merge base with the parent, else runs whole.
  A `card` scope is accepted only on `card` and `batch`.
- **Green record.**
  `lyx gate test` keeps a per-worktree record under `.lyx`, declared through `internal/lyxdirs`.
  A tier already green for the current clean tree takes no slot, runs nothing and prints a summary line marked carried; a partly covered tier runs only the remainder.
  A dirty worktree neither reads nor writes the record.
  The guard step writes entries under its own `-run` args.
- **Disjoint tiers.**
  Every test file carries exactly one `tierN` build tag; `integration`, `tmux` and `llm` become `tier1`, `tier2` and `tier3`, and untagged test files become `tier0`.
  An untagged test file holds only support code.
  `cmd/lyx/tiertags_test.go` enforces the rule.
  The guard step now runs guard tests per tier, under each tier's own command, and a `//lyx:guard` marker is allowed in `tier0` and `tier1` files only.
  Package loads in `internal/impactset`, `internal/planglyph` and `internal/planparser` list both the new and the old tag spellings.
- **Silent-tier check.**
  A `go` tier run fails when a package runs zero tests while holding a test file its tag selects, naming the tier, the package and `lyx config gate`.
  A run with `-run`, `-skip`, `-list` or `-bench` args is exempt.
- **Row entry gates.**
  A recipe row may carry `entry_gate`: one check or a list, each `module.key`, `module.key == literal`, `module.key != literal` or `budget.<Segment>`.
  Shed evaluates it when the run enters the row; a false gate records a `skipped` history entry with its reason and follows `on_done`, spending no budget.
  `lyx shed goto` bypasses the gate on purpose.
  Batten's Open-Window, Seed-IDE and Open-IDE rows moved from `switch:` to `entry_gate:`, and darn's recipe edge rewrite for a Code-Review budget of 0 is gone: darn's Describe row gets `entry_gate: budget.Code-Review`.
- **Darn tree scope.**
  `tree_scope_verify` is retired; tree scope is on exactly when `tree_scope` holds a tree.
  Reconcile migrates the hub file as long as the old key is present.
  A diff wholly under the trees renders `--gate <row key>` with the touched trees' targets.
- **`lyx gate report`.**
  It prints, per slug and gate key, tier runs, carried tiers, slot wait, wall time and launch time with its share.
  Go-json tier run records now hold a per-package launch list.
- **Docs.**
  The tier plan per recipe is stated in `internal/gateslot`'s doc.
  `PATTERN.md` gains `PATTERN-tier-tags`, `PATTERN-gate-carry` and `PATTERN-row-entry-gate`, and its other entries are shortened to keep the file within its headroom.
- **Follow-up.**
  The board entry `drop-transitional-tag-spellings` removes the `|| integration` term on guard-marked files and the old tag spellings after deploy.

## Design decisions to know

- Guard-marked files take `//go:build tierN || integration` so the installed `lyx`'s guard scan, which runs this task's gates until deploy, still accepts them.
- A carried tier trusts a pass over the same tree, tier declaration and args on this worktree.
  A flaky test that passed once is not rerun, and ignored files and inherited environment outside the tier's `env` are outside the check.
- The green record is a plain file; a forged whole entry makes later gates on that tree carry.
  This is accepted and stated in `internal/gatecli`'s doc beside the slot variable's bound.
- Shed's entered-row marker is cleared on every routed persist and skip, so a route back into the row evaluates its gate again, while a resume or a targetless relaunch re-calls it without evaluating.
- The measurement on OSL-1046 is `lyx gate report` run there after deploy; gate records are machine-local, and launch time comes from post-deploy runs only.

## To check by hand

- Deploy, then run `lyx config reconcile` on a hub holding `landing`, `final`, `round`, `guard_tier` and `tree_scope_verify`, and confirm the migrated keys and scopes, and that a second reconcile changes nothing.
- The operator's IDE seed needs `-tags=tier0,tier1,tier2,tier3` in gopls's build flags, or `tier0` test files leave the default view.
- Other branches landing with untagged or `integration`-tagged tests fail the one-tag scan until retagged.
- An old `lyx` on another machine refuses the reshaped `gate.yaml` until that machine deploys.
- The board entry `adopt-runs-tests` still names the retired `landing` list.
- `docs/benchmarks/test-suite-timing.md`'s tier figures before and after are not comparable, since `tier1` stops running `tier0`'s tests.
- The plan-level verify gate failed on `claudeengine.TestBuildSettings_MatchesGolden` and `cmd/lyx.TestCLIForm_EveryCommand`; two fix commits (`094e2ab1b`, settings golden carries the `tier2`/`tier3` shell-guard alternation; `621fe202e`, `lyx gate test` and `lyx batten run` Long pages stay within the 2000-byte cap) cleared them. Confirm both tests are green.
- Open review findings:
  - the relaunch paths' entered-row handling (targetless relaunch, relaunch with a target, target already in effect) has no test in shed;
  - the plan's tier-plan cards for the `fabricengine` and `shuttle` retags extend files that should be edited in place;
  - a duplicated forced-answer test sits in the batten consumer's tests;
  - the set of packages a retag or gate-key change affects was enumerated by hand rather than derived, so a package missed there is not caught.
