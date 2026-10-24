---
verdict: APPROVED
---

# Alignment report — umbrella-groundwork at parent-review

## Scope against the board entry

The decision record covers every part of the entry's goal: board project entries with an own-worktree flag and an ordered member list (D1, D2, D3, D15), the project chain as a new pair's parent (D4), fabric pairs from a named parent pair (D5), Worktree-Create forking from the task's beam (D7), batten and orch from other worktrees with batten's state under the prime (D8), notices to the beam's orch (D13), the create and convert verbs and the landing with no process while a beam lives (D9, D10, D11), the per-beam owner machine (D1, D4, PATTERN-owner-machine constraint), the folded `fabric-add-adopt-local` note (D6), the 2026-10-24 workspace-file decision (D12) and issue #724's slug rule (D14).

The design narrows or leaves out these points of the entry, each for a reason the entry's later decisions or the code supports:

- `lyx orch` runs only in the prime and in beams, where the entry's earlier Want says any drivable worktree. The later operator decisions make a task's parent its project's beam and the orch optional, so an orch in a plain task pair would have no tasks. Scope Out states this.
- `lyx beam promote` starts no orch thread, which follows the 2026-10-18 decision that keeps beam and orch apart.
- The beam landing uses a Go describe row and darn's Finalize, not Merge-In, Publish and PR-Gate. `loom-tail-review` has since removed those rows from darn.
- The ready-to-land handover through Bolt between machines is out of scope. No landing has this handover today, the prime's included, and Finalize's owner checks keep a beam's landing on the beam's owner machine. It is a hub-wide landing protocol, not part of the beam groundwork.
- #724's blocked-run cleanup is covered by `lyx batten cancel` (commit `c14db65e0`). Only `lyx batten cleanup`'s skip wording changes.
- `lyx adopt --all`, which the owner decision names, already exists (`internal/adoptcli/cli.go`).

## Other board entries

- `umbrella-daemon`: its 2026-10-24 decision points the beam commands `create` and `promote` here and keeps `start`, dobby, `draft`, the arbiter in the stop chain and the later removal of the ready notice. The design leaves all of those out and keeps the arbiter-free stop chain as it stands. The design neither duplicates nor contradicts it.
- `umbrella-anchor`, `loom-beam-darns`, `parallel-batch-parts`, `weaver`: the design leaves each of them out and gives them `AddOptions.Parent` and the beam module to build on. D10's promote ends the run, so `loom-beam-darns`' run-keeping conversion stays that entry's own work.
- `batten-window-default`: that note changes whether `--window` is the default, and D8 changes which reed session the window opens in. The two do not contradict each other.

No board entry is superseded: the notes this task folded in are already off the board.

## Overlaps with peers and the parent

- `run-profiles-ladder` (row 0 against own row 3, unknown order): both change Seed-Child in `internal/battenshed/seamchild.go`. The peer changes how a task's profile is resolved per recipe. This run adds recipe `beam` for a project entry, and a project carries no `profile` or `run_settings`. The peer's rule that a recipe without a default writes only the task's own settings covers `beam`, so the behaviours do not conflict. Both runs also edit the text of `internal/boardengine/doc.go`, `internal/boardcli`, `plugins/ly/skills/board/SKILL.md`, `contracts/stencils/orch/orch-template-role.md` and `docs/overview.md` in different places, which is a text-only overlap. The landing order is unknown, since the peer's row is 0.
- `dotnet-scoped-verify` (row 3, tie): it has no changed paths and no decision record yet, and its .NET verify scope touches nothing this run changes. No overlap.
- `fork-git-guard-scratch-repo` (row 16): it changes only `internal/burlerengine`. No overlap.
- Parent commit `23487316c` (gate test vet step for a `--card` scope): it changes only `lyx gate test`. No overlap.

The PATTERN entries the record cites or changes (`PATTERN-cwd-resolution`, `PATTERN-pair-teardown`, `PATTERN-owner-machine` and the others in its Constraints) are all in the current `PATTERN.md` after `04aafa88c`, and no peer changes them.
The new `PATTERN-pair-parent` index line and the `beam` module line in `docs/overview.md` are shared bookkeeping and not a conflict.
