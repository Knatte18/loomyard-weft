# Support log — verify-publish-hardening

## Interview

Autonomous mode: no operator answered.
Every question below was settled by an auto-pick after reading the board entry, the related entries (`pattern-init`, `pair-lifecycle`, `review-finding-class`, `parent-review-hardening`) and the code.

1. **Where does the per-card integration check come from?**
   The board proposed Plan-Write appending `go test -tags integration <pkg>` and `go test ./cmd/lyx/` to card `verify:` lines.
   Measured here: untagged `go test ./...` takes ~13 s cold and ~2 s cached, so the whole untagged suite per card covers callers and the `cmd/lyx` repo scans cheaply.
   Auto-pick: Go derives the card gate at `begin-batch` (D1), plus a Plan-Write grep rule for reworded messages (D2).
2. **Does a re-run after a fix hit Go's test cache?**
   Measured: `go test -tags integration` over `verifyrun`, `landingshed`, `gitexec`, `hubforge`, `battencli` and `reedcli` reported `(cached)` on the second run (49.8 s → 1.2 s).
   Failed results are never cached, so a re-run re-executes only failing and changed packages.
   Auto-pick: the gate re-runs the full command each attempt; no separate failed-packages-first step.
3. **Why did bisect name card 07 for card 09's line?**
   `accumulatedCardSHAs` lists SHAs in sequenced execution order, while `begin-batch` does not enforce that order, so list order need not match ancestry.
   `bisect` also assumes every intermediate card SHA passes the integration suite; a card adding or editing a test can legitimately precede the card that makes it pass, and the regressing identity then first fails at the earlier card.
   The package doc also records that bisect runs on pre-merge trees and attributes a mid-run parent merge's regression to the next card.
   Auto-pick: bisect goes; a touch-based hint survives (D4).
4. **What happens to triage?**
   The flaky rerun is cheap under the cache and stays.
   The baseline run checks out the start commit detached in the live worktree and only defers the halt, since Publish's verify fails on a parent-side failure anyway.
   Auto-pick: drop the baseline (D3, D4).
5. **Who fixes a gate failure?**
   The board's direction: Merriam forks a fixer with full plan context.
   Auto-pick: one background fixer fork per failure, from a Go-rendered stencil, with Go checking its commits (D5).
6. **Merriam's polling.**
   `9ae921b24` removed the batch and integration polling.
   The `recover-batch` re-call loop and the "started non-interactively" framing remain.
   Shuttle reads backgrounded Bash as `EventWaiting`.
   Auto-pick: background `recover-batch`, blocking to a terminal digest (D6).
7. **How does Publish know the merge brought nothing new, and how is a crash covered?**
   Auto-pick: a verified-tree record keyed on HEAD's tree SHA and the command, written only after a pass, shared by every verify site (D7).
   It replaces the pending marker.
8. **How does the perch get a test gate?**
   Burler rows already take `gates:`.
   Auto-pick: a `verify` gate entry on Webster-Burler over D7, which skips when the round changed no code (D8).
9. **Where did Publish's dirty tree come from?**
   `mergeresolve` stages only the conflicted paths, so any other edit by the conflict session stays uncommitted under verify and push.
   Auto-pick: stage tracked edits into the merge commit, refuse untracked files (D10), and check cleanliness around every step (D9).
10. **The conflict-report reuse.**
    `buildConflictSpec` names `conflict-resolution-r<attempt>.md`, and the spec validator refuses an existing file.
    Auto-pick: clear old reports at `Resolve` entry (D11).
11. **Prompt delivery.**
    `claudeengine.Prepare` already writes `prompt.md`; only the launch argument carries the whole prompt.
    Every role launches through shuttle.
    Auto-pick: a provider-level pointer for every fresh launch (D12).
12. **Strand role and model key.**
    Auto-pick: strand role `webster`, model key stays `master` (D13).
13. **`activity.wait` while the gate runs.**
    `composeActivity` sets `Wait` only for blocked, failed or awaiting states, while loom already has a live `Hooks.Waiting` note for running producers.
    Auto-pick: a running marker read by the `Waiting` hook (D14).
14. **`begin-batch`'s model assertion.**
    Auto-pick: keep it; it is dormant and the board names no gain from removal.
15. **Gate Self-Check Parity for the new gates.**
    Auto-pick: `lyx webster verify` (D15) and a new Verified-Tree Invariant (D16).

## Rejected alternatives

- **Plan-Write writes per-card integration `verify:` lines** (the board's first suggestion): relies on an LLM remembering, needs a plan-gate check to enforce, and duplicates what the spec's Tier 1 already designs as mechanical.
- **A `webster.yaml` list of scan packages run per card**: repo-specific config the loomyard hub would have to set by hand; the whole untagged suite costs seconds and covers it.
- **Go re-runs the card gate at `record-batch`**: `record-batch` is Merriam's foreground call, and an uncached integration run there risks the auto-background threshold; the webster gate backstops.
- **Keep bisect as a hint in the gate findings**: its evidence is unreliable for the structural reasons in Interview item 3, and it moves HEAD in the live worktree.
- **Keep the triage baseline (pre-existing classification)**: one more full-suite run on a cold tree plus a detached checkout, only to defer a halt Publish's verify makes anyway.
- **Keep the separate integration-fix strand**: a cold session without plan context; the board replaces it with Merriam's own fork.
- **Merriam fixes in-session**: breaks Merriam's never-edit rule and pollutes its context.
- **Pending marker plus a commit-SHA record**: the tree key subsumes both, and also covers rebases and no-op merges.
- **Store the verified record in `_lyx`**: every pass would cause a weft commit; losing an ephemeral record only costs a re-run.
- **Refuse any conflict-session edit outside the conflicted paths**: halts for a human on a semantic-conflict fix the session already made correctly.
- **Persist `activity.wait` mid-producer from shedengine**: adds a status-write path from inside a producer call, while a live hook already exists.
- **Rename the `master` model key to `webster`**: needs alias handling in config loading for no behavior change.
- **Pointer only for prompts over the limit**: two launch shapes to test, and a role near the limit flips between them.
- **Remove `begin-batch`'s model assertion**: touches state, docs and a sandbox scenario for no runtime gain.

## Review rounds

_No rounds yet._

## Question ledger

| # | Question | Status | Resolution |
|---|---|---|---|
| 1 | Card gate source: Plan-Write or Go? | resolved (auto-pick) | Go-derived, D1 |
| 2 | Scan packages per card? | resolved (auto-pick) | whole untagged suite, D1 |
| 3 | Reworded messages | resolved (auto-pick) | Plan-Write grep rule, D2 |
| 4 | Does the test cache survive integration re-runs? | resolved (measured) | yes; full re-run per attempt |
| 5 | Why the bisect misattributed | resolved (code reading) | list order ≠ ancestry; intermediate SHAs not green |
| 6 | Bisect survives as a hint? | resolved (auto-pick) | no; touch hint, D4 |
| 7 | Triage baseline | resolved (auto-pick) | dropped, flaky rerun kept, D3 |
| 8 | Gate attempt cap | resolved (auto-pick) | `verify_gate_attempts: 3`, D3 |
| 9 | Fixer mechanism and its bound | resolved (auto-pick) | one fork, commit check, D5 |
| 10 | `recover-batch` polling | resolved (auto-pick) | backgrounded blocking call, D6 |
| 11 | Crash-safe landing verify | resolved (auto-pick) | verified-tree record, D7 |
| 12 | Perch verify gate | resolved (auto-pick) | Webster-Burler `verify` gate, D8 |
| 13 | Dirty tree at Publish/Finalize | resolved (auto-pick) | clean checks, D9; stage tracked edits, D10 |
| 14 | Conflict report reuse | resolved (auto-pick) | clear at `Resolve` entry, D11 |
| 15 | Prompt delivery | resolved (auto-pick) | provider pointer, D12 |
| 16 | Strand role and model key | resolved (auto-pick) | role `webster`, key `master`, D13 |
| 17 | Gate visibility | resolved (auto-pick) | `Waiting` hook marker, D14 |
| 18 | `begin-batch` model assertion | resolved (auto-pick) | kept |
| 19 | Self-check verb and invariant | resolved (auto-pick) | `lyx webster verify`, D15; Verified-Tree Invariant, D16 |
| 20 | Pre-existing parent failures | open risk | consume gate attempts, then halt |
