# Support log — bugfix-fabric

## Interview

Autonomous mode: no operator answered; every question below was auto-picked from the board entry and the code.

1. Scope. The board entry holds three bug sections and states bugfix only, no new verbs, config keys or features, with `fabric-add-adopt-remote` in scope whole by operator decision of 2026-10-06. No other board task overlaps: `bugfix-sessions` and `bugfix-webster` cover shuttle/loom/webster, and the `config-overlay` and `orch-notice-channel` notes are unscheduled and unaffected.
2. `.weft/` creation. Read `ReadOriginFor`, `ensureWeftLockDirAt`, and the weft worktree creators in `weftwiring.go`, `add.go` and `reconcile.go`. Only writers create `.weft/` today; `adoptWeftWorktree` and the fork helpers do not. Picked: create it at every weft worktree birth (D1), heal it in reconcile (D2).
3. Silent no-parent. `ReadOriginFor` answers "no record" when the lock directory is absent. Picked: an error naming the gap and `lyx fabric reconcile` (D3), leaving `Remove`'s error-ignoring read as is.
4. Config reaching the parent. Both cited incidents are obsolete in form (`board.yaml` is HubWide now; PATTERN moved to warp). Per-pair tuning of `loom.yaml` is a legitimate hub practice, so a blanket carry is harmful. Picked: Finalize detects per-worktree config changes since the fork point and queues an orch notice (D4).
5. Adopting origin branches. Read `Add`'s pre-flight leftover probes, `reconcileMissingWeft`, `switchOrForkWeft`, and `clone`'s `suffixWeftPrimaryBranch`. Picked: one local-then-origin resolver (D5), a live-pair rule in `add` that ties warp adoption to a live weft (D6), and origin checks in `reconcile` and `checkout` (D7).
6. Testing. The board requires CLI-wiring regression tests. Picked: hubforge fixtures with a bare origin, `integration` tier, driving the cobra commands (D8).
7. Sequencing, added by the parent on 2026-10-06. Plan-Write writes the plan now against the current tree and names test files by path (D9). Webster starts only after test-suite-prune-core and test-suite-prune-rest have both landed on main; then the pair is synced with main and the plan's test targets are re-validated, re-planned only where a prune renamed or deleted them. The operator runs Discussion and Plan while the prunes run and holds implementation until they land, since prune-rest rewrites the fabric tests this task extends.

## Rejected alternatives

- Landing carries `_lyx/config/` changes into the parent's weft with a conflict stop: pushes per-pair tuning (a small task's `review_max_bounces`) into the parent, and cannot tell a parent-meant change from a pair-local one.
- Task-made config changes declared in the PR description by the Describe agent: depends on an agent noticing, which is the same silence in another place, and Publish runs only when the parent needs a PR.
- Finalize Stuck on any config change: clearing it needs an acknowledge verb, which the scope forbids.
- Finalize Stuck on a failed config-diff read: the notice protects no correctness of the landing, so halting breaks the loom rule that a non-correctness guard warns and records, and departs from Finalize's own rule that a git fault is an error and never Stuck.
- `ResolveParent` creating `.weft/` itself and resolving the parent: a write in a read path the docs promise creates nothing, and the board asks for the gap to be named.
- `ReadOriginFor` reading the record lockless when `.weft/` is absent: hides the gap instead of naming it.
- A new `ReconcileAction` for the lock-directory repair: widens the envelope vocabulary in a bugfix.
- Adopting `origin/<slug>` in `add` whenever it exists, regardless of the weft: resurrects a removed pair's stale warp branch (squash landing makes every landed branch diverge from HEAD).
- Forking warp from HEAD when the live pair's origin warp is an ancestor of HEAD: contradicts "never creates a new local branch in its place" for no gain.
- Falling through to a fresh fork when the origin probe fails: that is the reported bug.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): Is a Finalize notice an allowed "fix" under no-new-features? Yes: it adds no verb, flag or key, and fixes the reported silence. See D4.
- Q2 (auto-pick, resolved): Which orch gets the notice for a stacked task? The prime's notice queue, as batten's notices. See A2.
- Q3 (auto-pick, resolved): Does `reconcile` adopt an archived origin weft? Yes, because a live warp worktree on that branch is present. See D7.
- Q4 (auto-pick, resolved): Fast-forward a local weft branch behind origin in `add`? Yes; divergence still refuses. See D6, A5.
- Q5 (auto-pick, resolved): A failed config-diff read in Finalize: warn or Stuck? Warn, land, and queue a notice that the diff could not be read, so nothing is lost silently and no landing halts for an advisory output. See D4.
- Q6 (auto-pick, resolved): Key-level or file-level config report? File-level with fork-point and tip SHAs, so the hub can diff; key-level parsing adds code without changing what the hub does. See D4.
