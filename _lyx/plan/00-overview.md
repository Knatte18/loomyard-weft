---
format: 5
approved: true
language: go
---

# Plan: bugfix-sessions

Fix the session-lifecycle, notice and halt bugs the board entry `bugfix-sessions` collects, each with a regression test that drives the real CLI wiring.
Typed prompts are paced and confirmed submitted; an escalation halt spawns no reflection; a re-run producer and a finished or reset Webster recovery remove the strand they supersede; a clean step handoff never reads as a crash, and a crash-resume note shows its evidence; the orch watcher reloads only after a fresh compaction, types `/reload-plugins` first, retypes skills only after `/clear`, and `lyx orch start` waits for a stopping watcher; a Burler review with broken YAML is repaired in its own session, with its self-check verb; and neither `lyx loom start` nor `lyx orch start` moves a tmux client.

## Card Index

1 — paced-text-submit — Claude text-plus-submit sequences type the text, settle, then send Enter as its own step
2 — claude-input-box-reader — add the InputBoxReader provider capability and its Claude implementation
3 — send-confirms-submission — sendVerified presses Enter again, at most twice, while the box still holds the sent text
4 — escalation-halt-skips-reflection — an escalation-to-human halt writes its note and spawns no reflection
5 — rerun-removes-superseded-strand — the attach probe removes live respawn-eligible strands of the same outputs before a fresh run
6 — recover-batch-removes-terminal-strand — recover-batch removes its recovery strand on every terminal outcome
7 — reset-removes-recovery-strands — webster reset --to start removes every live recovery strand first
8 — step-error-keeps-handoff-voucher — a step that returns an error after persisting still records the handoff voucher
9 — crash-resume-note-evidence — the crash-resume note names the lock and voucher readings and lists all history
10 — compaction-boundary-turn-ends — the provider reports turn ends after a compaction boundary and whether the read one is newest
11 — watcher-reloads-fresh-compaction-only — the watcher reloads only after a compaction with exactly one, read, turn end after it
12 — reload-plugins-sequence — add the /reload-plugins provider sequence and Runner.ReloadPlugins
13 — orch-reload-sequence — every reload types /reload-plugins first; only a /clear reload retypes skills
14 — loom-start-never-attaches — lyx loom start never attaches or switches a tmux client
15 — orch-start-never-attaches — lyx orch start never attaches or switches a tmux client
16 — orch-start-waits-for-stopping-watcher — a signalled watcher records stopping and orch start waits for it within a bound
17 — burler-review-parse-gate — every burler round gates its review file on ParseReview, re-prompting the reviewer
18 — burler-validate-review-verb — add lyx burler validate-review, the review gate's self-check verb

## Shared Decisions

### Decision: provider-seam

- **Decision:** input-box reading (Card 2), the submit pacing (Card 1), turn-end counting after a compaction (Card 10) and `/reload-plugins` (Card 12) live in `internal/shuttleengine/claudeengine`; `shuttleengine` declares only provider-neutral capability types and `orchengine` consumes them through its `Session` seam.
- **Rationale:** `PATTERN-shuttle-provider-seam`; `PATTERN-orch-pane-single-writer` keeps every orch typing in the watcher, idle-gated.
- **Applies to:** cards 1, 2, 3, 10, 11, 12, 13

### Decision: regression-tests-through-the-wiring

- **Decision:** each fix's regression test enters through the cobra verb or the CLI-level wiring that reaches it (`Runner.Send` behind `lyx shuttle send`, the generic step body behind `lyx loom step`, `Watcher.Tick`, `RecoverAwait` behind `lyx webster recover-batch`, the cobra `reset`, `start` and `validate-review` verbs, `Engine.Run` behind the Burler row), with fakes only at the tmux, provider and LLM substrate, in the lowest tier the substrate allows, extending an existing test where one covers the path.
- **Rationale:** D14; `PATTERN-test-speed`, `PATTERN-test-economy`, `PATTERN-test-isolation`, `PATTERN-testkit`.
- **Applies to:** all cards

### Decision: test-targets-revalidated-after-the-prunes

- **Decision:** cards name production files as their main targets and test files by path; once test-suite-prune-core and test-suite-prune-rest land and the pair is synced with main, the test targets are re-validated and re-planned only where a prune renamed or deleted them.
- **Rationale:** the parent's post-Discussion decision of 2026-10-06; Webster starts only after both prunes land.
- **Applies to:** all cards

## verify:

go build ./...
go test ./...
go test -tags integration ./...
go test -tags tmux ./internal/loomcli/... ./internal/webstercli/...
go vet -tags llm ./...
