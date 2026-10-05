---
format: 5
approved: true
language: go
---

# Plan: webster-audit-recovery

A Webster run that halts on its run-exit audit or its verify gate becomes recoverable by the agent that receives the refusal, without the operator and without discarding the run's batches.
The plan closes the four `board-model` failures:
shuttle's gate waiting forever on a hung background shell (cards 1–5), with the hung-stdin class closed at its source (card 4);
a fork-contract-write finding `accept-audit` could never accept (cards 6, 7, 10, 13);
garbled way-forward text (cards 11, 14);
and recoveries that name a command the agents are denied, replaced by a guarded `lyx webster reset` verb and a PATTERN scan (cards 8, 9, 11, 12, 15).

## Card Index

1 — shuttle-background-tasks — claudeengine reports a waiting turn end's outstanding tasks as a neutral fork/shell list
2 — shuttle-shell-wait-config — `background_shell_wait_min` joins shuttle.yaml, non-positive refused
3 — shuttle-shell-expiry — the wait loop counts a turn end once its non-awaited shells outlast the bound
4 — claude-bash-stdin-hook — a PreToolUse hook gives every agent Bash call `/dev/null` as default stdin
5 — webster-expired-shells — Master awaits `recover-batch`; a waited-out shell becomes a warning, summary section and friction note
6 — webster-contract-file-stencils — forks are told never to write the contract files; Master needs no script
7 — shuttle-write-events — the audit facts carry each write's time and result
8 — fabric-reset-pair-warp — a gated fabric primitive resets a pair's warp checkout, exempting the run's own dirt
9 — webster-prefix-head-persist — the verify gate persists and clears its pre-fix head in state.json
10 — webster-run-writes — one reader of every Master and fork write the run's sessions record
11 — webster-reset-plan — Webster resolves a reset target and refuses unsafe resets before fabric is called
12 — webster-reset-verb — `lyx webster reset --to start|pre-fix`, with its refusal-spec rows
13 — webster-contract-file-evidence — a fork-contract-write finding clears on absence or Master's later write
14 — webster-audit-way-forward — audit and fresh refusals name each finding once with reasons and a numbered command list
15 — webster-no-denied-recovery — the remaining reset texts name the verb, and PATTERN-no-denied-recovery scans for denied commands

## Shared Decisions

### Decision: D3 takes the primary route

Claude Code's hook documentation ("PreToolUse decision control", code.claude.com/docs/en/hooks) states that `updatedInput` is allowed when `permissionDecision` is `allow` or omitted, and that an omitted decision leaves the normal permission flow in charge.
The hook omits the decision, so it grants nothing; D3's fallback (denying interpreter shapes) is not selected.
Whether Claude Code matches allow and deny rules against the prefixed command as it did against the original is not stated there; card 4's smoke test pins it live, and a failure there is the trigger for D3's fallback.

### Departure from D7, for operator confirmation: the report-head and last-batch-head moves name `git reset --keep`

D7 says every site that moves HEAD back with git names `lyx webster reset`.
D6 and A6 fix the verb's targets to `start` and `pre-fix`, and A6's premise that every Webster refusal needs one of the two does not hold: record-batch, recover-batch and accept-audit also refuse toward moving HEAD back to the report's `head_sha` or the last batch head.
The plan does not widen the verb beyond its decided bound.
Card 15 instead names the plain `git reset --keep <sha>` at those sites: it is not on the deny list, it refuses rather than discards uncommitted changes, and refusal-spec rule 3 admits a plain git command no lyx verb covers.
Card 12's standalone-mode refusal uses the same command, since standalone has no pair for the fabric gate to guard.
This is a departure from D7 the record does not grant, so the operator confirms it or widens the verb's targets; D7's verb rule is carried unchanged for every `start` and `pre-fix` site.

### Decision: D10 lands with the last rewording card

D10 asks for the PATTERN entry in the same commit as D6 and D7.
The verb (card 12) and the rewording (cards 14 and 15) are separately committable cards, so the entry and its scan land in card 15, the commit that completes D7, which keeps CLAUDE.md's "invariant with its code" rule: the scan passes from its first commit.

### Decision: one re-entry step

A pending-findings or run-exit way forward ends in exactly one re-entry step: `RunDeps.ReentryStep` (`lyx webster run` from the CLI, "re-step the <row> row" from the shed adapter) on the accept-audit route, and `lyx webster run --fresh` on the reset route, which the shed adapter never runs itself because its `Fresh` is fixed false.
The shed adapter stops appending its own step to the reason (card 14).

### Decision: sandbox coverage is module-level

`PATTERN-sandbox-coverage` keys on modules, and the webster module is already covered by its sandbox scenario, so card 12 adds no scenario.

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```
CGO_ENABLED=1 go build ./...
CGO_ENABLED=1 go test ./...
CGO_ENABLED=1 go test -tags integration ./...
CGO_ENABLED=1 go vet -tags smoke ./...
```
