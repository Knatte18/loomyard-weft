# One-repository sandbox trial 2: tasks run to landing and abandoned

Adds the report `docs/research/single-repo/sandbox-2-tasks-2026-11-21/report.md`, the second of three trials of the one-repository hub layout from 57015a52.

Five darn tasks, each a few lines of Go in the sandbox's `greet` package with recipe darn, profile light and writer opus medium, ran in `/home/knatte/Code/lym-sandbox-onerepo-LYMHUB`:
two one after the other, two in flight at once, and one started and then cancelled with `lym batten cancel --remove` and removed from the board.
Four landed; the abandon left no pair.

Origin matched the layout at every checkpoint the report tabulates:

- A run in flight has its code branch under `refs/heads/`, which GitHub lists, and its records branch under `refs/lym/heads/<slug>-weft`, which GitHub does not.
- About a minute after each landing, teardown deletes both branches and pushes the records head as `refs/lym/tags/archive/<slug>/<sha12>`; GitHub's tag list stayed empty.
- The abandon ends in the same state as a landing.
- At the end, origin's branch list is `main` alone, origin's board, `main-weft` and `main` equal the local tips, and the code store holds no records ref.

One lym bug, filed as note B2 `rules-read-plan-gated-403` on this hub's board:
the first landing halted at Finalize because GitHub answers the branch-rules read of a private free-plan repository with 403 "Upgrade to GitHub Pro",
and `githubHaltReason` in `internal/landingshed/finalize_pr.go` reports every 403 as a bad token, naming a way forward that cannot work.
The orch had set `pull_request` off while the run held pull request #1, which sent Finalize to that read.

The report names one gap: the trivial concurrent task asked for a package doc comment the seed already had, so it landed no change,
and the concurrent pair shows two runs in flight and landing one after the other but not a second code change merged onto a freshly moved parent.

## Design decisions

- The orch filed, released, chained and abandoned the sandbox tasks: the sandbox board refuses a task session both a `kind: task` upsert and a profile or run_settings field, and starting the sandbox dobby is the orch's.
- The five runs used the whole cap, so the empty `greet-doc` landing was not rerun; the report leaves the gap to trial 3.

## Merge danger

**Door:** two-way
**Blast radius:** docs
