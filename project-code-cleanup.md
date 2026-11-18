# P1 Code cleanup: cut and simplify lym code and tests

Rendered 2026-11-18 17:50 UTC; every age below is as of then.
Under: [Board](README.md).

[`code-cleanup`](design-code-cleanup.md) · Age 26m · Owner `neve`

Make lym smaller without changing behaviour: fewer tests, leaner design, no dead compatibility code, no unused features

Make lym smaller without changing what it does, one mechanical pass at a time.

Background: `docs/research/rewrite/shrink-estimate-2026-11-18/report.md` found almost no copy-paste duplication (under 1 %).
The size sits in tests (about 2 test lines per code line), comments,
near-identical helpers a duplicate detector misses, design that makes every caller handle more than it needs,
compatibility and migration code, and features nobody uses.
A clean-room rewrite was judged too large for the gain (2026-11-18); this project cuts in place instead.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Find real uses for the per-test coverage maps**<br>• A research run on Fable searches the web and the code for what per-test coverage maps can do for lym and its agents, and ranks the uses<br>• `darn: writer fable[high]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`testmap-use-cases`](design-testmap-use-cases.md) | darn · docs | PR-Review | neve | 21m<br>21m | testkit, gate |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | `draft` **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | darn · solid |  | 8d<br>26m | testkit, gate |

## Notes (4)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E3 |  | **Comments: check what the big comment rewrite left, before any new pass**<br>• Last week a run of nearly 3,000 agents rewrote docstrings and doc files; measure what is still long before deciding on more | [`comment-trim-second-pass`](design-comment-trim-second-pass.md) | 26m<br>26m | docs |
| E5 |  | **Design: simplify interfaces that make every caller handle too much**<br>• For example one result type instead of a four-value return handled at 60 call sites; find such spots per module and fix them one module at a time | [`design-simplify-callers`](design-design-simplify-callers.md) | 26m<br>26m | fabric, landing |
| E6 |  | **Remove compatibility and migration code no hub needs any more**<br>• Retired names, key renames and one-off data migrations still in the code; keep the cheap refusals that point to the new name, delete the rest | [`drop-compat-migration-code`](design-drop-compat-migration-code.md) | 26m<br>26m | fabric, loom, board |
| E7 |  | **Remove features nobody uses**<br>• Find unreachable code mechanically and list features with no recent use; the operator decides which go | [`drop-unused-features`](design-drop-unused-features.md) | 26m<br>26m | tools |
