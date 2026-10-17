# Landing trailer block: trailer_text plus one line per engine that ran

Finalize used to append exactly one `Co-Authored-By: <value>` trailer to the landing commit, with the value taken from `landing.yaml`'s `co_authored_by`.
That gave the operator no say over what the trailer block holds.
The landing commit's trailer block is now built from two operator-set keys, and each line is copied verbatim with no prefix added.

## What changes

- `landing.yaml` drops `co_authored_by` and gains two keys, both `CategoryCrossPair`:
  - `trailer_text` holds fixed lines that land on every commit, for example `Created using Loomyard`.
    It may span several lines.
  - `engine_trailers` maps an engine name to its trailer text.
    It is an open map, so reconcile and the layer stack carry the operator's entries whole.
- The template default keeps today's commit: `trailer_text: ""` and `engine_trailers.claude: "Co-Authored-By: Claude <noreply@anthropic.com>"`.
- `Config.TrailerBlock` builds the block in this order:
  - the trimmed `trailer_text`;
  - then the trimmed `engine_trailers` text of each engine that ran, in sorted engine-name order, one line each.
- What adds no line:
  - an empty or whitespace-only value;
  - an engine with no entry;
  - an engine that did not run.

  With both keys empty, the commit carries no trailer block.
- `summaryparser.Summary.LandingMessage` takes the composed block and appends it after a blank line.
  An empty block yields the bare commit message.
- Finalize reads the engines through a new `landingshed.Deps.EnginesRun` seam.
  The read happens next to the change-description parse, before any parent-side mutation.
  If the read fails, Call returns an error and nothing is merged.
  `internal/loomcli` wires the seam to the run's shuttle runner.
- `shuttleengine` keeps a record of which engines ran:
  - Start adds each run's engine name to `engines-run.json` at the run directory root before it registers the strand.
  - A failed ledger write fails the start and removes the run directory.
  - `Runner.EnginesRun` returns the sorted union of the ledger and the engine of every run record still under the root.
    A record that names no engine counts as the engine set's default.
- `LoadConfig` refuses an `engine_trailers` key that names no engine `modelspec` knows.
  The error names the key and lists the known engines.
- `landingshed.MigrateConfig` is now landing's `Migrate` hook, so `lyx config reconcile` rewrites `co_authored_by: X` into `engine_trailers: {claude: "Co-Authored-By: X"}`.
  If the document already holds `engine_trailers`, that map is kept and `co_authored_by` is dropped.
  A second pass changes nothing.
- The Describe validation still refuses a `Co-Authored-By` line in the change description.
- Docs updated in this change:
  - `internal/landingshed/doc.go` and `internal/shuttleengine/doc.go` describe the trailer block and the ledger;
  - `docs/overview.md` and `contracts/specs/loom-final-summary-spec.md` describe the new message composition and keys.

## Design decisions

- **Where "engines that ran" comes from.** Shuttle sweeps a per-run directory once its strand is gone, so run records alone cannot answer the question at landing time.
  The ledger is a file beside the run directories, and the orphan sweep removes only directories, so the ledger outlives every run in the worktree.
  Every runner in a task worktree anchors at the same run directory root, so the ledger lists the engines of the task's own sessions.
  It is never tracked and goes away with the worktree.
- **Runs started before the ledger existed.** The scan of surviving run records covers sessions started by an older binary.
  A run already in flight across the upgrade can still miss an engine whose only sessions were swept before the ledger existed.
- **Typos.** An unknown engine key in `engine_trailers` is refused at load, because a typo would otherwise drop that engine's line silently.
- **Line order.** Fixed lines come first and engine lines follow in name order, so the message is stable across runs.
- **Test helpers.** The shuttleengine tests that asserted "no leftover entries" under the run directory root now count run directories only, since the ledger and its lock file are legitimate files there.

## To check by hand

- On a hub with a hand-edited `co_authored_by`, run `lyx config reconcile --apply` and confirm `landing.yaml` holds `engine_trailers.claude: "Co-Authored-By: <that value>"`, an empty `trailer_text`, and no `co_authored_by`.
