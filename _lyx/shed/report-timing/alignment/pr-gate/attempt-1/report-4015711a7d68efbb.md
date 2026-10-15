---
verdict: APPROVED
---

## Alignment at PR-Gate: `report-timing`, pull request 647 at `eb48bc8d2`

The diff against `main` (merge base `474fb6680`) matches the board entry's goal and the decision record.

- The verb `lyx report timing [--since] [--by day|week] [--format json|markdown]` is added beside `convergence` in `internal/reportcli/timing.go`, prime-only through the shared `resolvePrime`, and computes in `internal/reportengine` over the same `Sources` snapshot, which gains only the records branches and the snapshot time.
- Per-run phases, spans, waits by kind, sleep, arbiter and alignment attempts, day and week buckets and stops by settler are in `timing.go`, `timingspans.go`, `timingwaits.go`, `timingstops.go` and `timingmarkdown.go`, as the decisions lay out.
- `convergence`'s output is unchanged: its reads gain only fields the timing report consumes (seed `started_at`, batten record commit, decoded alignment attempts), and its integration test keeps its assertions.
- The caller label is classified in the CLI layer (`loomcli` and `battencli` `arm.go` through `agentname.CallerLabel`) and passed as a string; `shedverbs` and `shedengine` read no environment and gain no import.
  The Go driver and batten's publish retry pass `shed`; the hidden `goto --by` admits `shed` alone, as plan card 6 states, with its refusal added to `contracts/specs/refusal-spec.md`.
- The package moves `internal/durfmt`, `internal/sleepjournal` and the settle-record format into `internal/shedrun` are in, with `PATTERN-leaf-packages` entries, the overview's module table, the sandbox suite questions and the `test-budget.yaml` rows.
- `reportengine`'s allowlist gains `internal/durfmt` only, per `PATTERN-told-geometry`.
- Commit `a993bef03` (records branches parsed through `fabricengine.WeftWarpSlug`) is reverted by `0fb30f623`; the branch is still named through `fabricengine.RecordsBranchName`, so `PATTERN-fabric-git` holds.
- Out-of-scope items stay out: no token totals, no `spawned_at` writer repair, no change to `convergence`'s output, no hub-pause accounting.

The carried-over risks were read: the unreliable alignment `spawned_at`, power-off time counted as run time and older runs reading `unknown` are recorded open risks, not deviations; the Discussion-Review and Plan-Review carry-overs concern design detail the code settles (batten rows before and after `Run-Shed`, `Run-Shed` itself by its child's producer).

## Overlap with peers and parent commits

`lyx loom peers` lists no parent commits; this run's `own_row` is 22.

| Peer | Row | What overlaps | Landing order |
|---|---|---|---|
| `creel-core` | 12 | `internal/agentname/agentname.go`: this run adds `CallerOperator` and `CallerLabel` for the history caller label; creel adds non-strand sender names for message identity (its D4). Different behaviours at different places; this run does not touch `PATTERN-agent-name`. | `report-timing` lands first; `creel-core` takes it at Merge-In |
| `engine-registry` | 12 | `cmd/lyx/testdata/test-budget.yaml` and `docs/overview.md` module-table lines at different places (bookkeeping, text only) | `report-timing` lands first |
| `sibling-overlap-check` | 12 | `cmd/lyx/testdata/test-budget.yaml` (bookkeeping) | `report-timing` lands first |
| `reed-bindings-repin` | 17 | `cmd/lyx/testdata/test-budget.yaml` (bookkeeping) | `report-timing` lands first |
| `plan-test-file-path` | 7 | none | — |

Board entries that touch this run, none in flight: `engine-usage-readers` will rewrite `tools/tokencount` onto the moved `internal/sleepjournal`, as the decision record's open risk names; `shed-arbiter-all` will edit the one arbiter reader; `orch-split`'s later run driver gets its own label, as The caller label decision bounds.
No board entry is made obsolete by this run.
