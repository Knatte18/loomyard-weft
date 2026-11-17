---
format: 5
approved: false
language: go
---

# Plan: each launch writes its manifest, a parity test compares the engines per seat, and lym shuttle manifest prints both

Each engine's `Prepare` writes a launch manifest: one item per thing the session receives, keyed by channel and an engine-neutral key.
The record goes into the run directory, and a launch keeps a copy under the run-directory root's `manifests` directory.
A parity test in a new `internal/shuttleparity` package builds one spec per engine for every seat and every other launch-spec producer, runs both `Prepare`s on fixtures and fails when the Codex manifest lacks a compared item of Claude's.
A scan fails on a launch-spec literal or seat table the test does not list.
`lym shuttle manifest` prints both engines' manifests for a seat, and the board carries the bench rule.

## Card Index

1 — heddle-standing-keys — standing passages and heddle file entries carry their slug and the engine they are keyed to
2 — manifest-record — the manifest record type, its comparison, the copy a launch keeps and the scans that skip it
3 — claude-manifest — Claude's Prepare writes its manifest
4 — codex-manifest — Codex's Prepare writes its manifest
5 — engines-with-seams — build the engine set with a given hook runner and sandbox probe
6 — seat-parity — the parity package, its seat catalog and the per-seat parity test
7 — producer-table — the producer table and the scan for unlisted launch-spec literals and seat tables
8 — shuttle-manifest-verb — lym shuttle manifest prints each engine's manifest for a seat
9 — pattern-engine-parity — the PATTERN-engine-parity entry and its background
10 — bench-rule — the bench rule on the Codex project and the writer bench

## Shared Decisions

### Decision: items are read back from the written artifacts

D1 reads the manifest back from the launch: each engine lists every candidate item with the exact text or hook command its delivery writes, then keeps an item only when the artifact as written holds it.
For Claude the artifacts are `settings.json`, the appended system prompt and the launch line; for Codex, the launch line with its developer instructions and hook settings; for a heddle file entry, the marked `CLAUDE.local.md` or `AGENTS.override.md` it renders into.
An item therefore exists only where its delivery reached the launch, so dropping a delivery anywhere in `Prepare` drops its item, which is what the test needs.

### Decision: the edit directive is read off the rendered prompt by shuttleparity

An engine is told no stencils directory, so it cannot tell its edit-directive copy in a prompt.
`shuttleparity.PromptItems` adds the `edit-directive` item from the prompt rendered for each engine, in the test and in the verb alike; the run record lists the prompt engine but not the directive.

### Decision: the copy is written by the launch step that runs Prepare

`Runner.launch` copies the record once the run record is saved, so only a launch whose strand registered keeps a copy.
D1's "a launch's Prepare also writes a copy" is met for every launch, and `lym shuttle manifest`, which calls `Prepare` directly, keeps no lasting copy, as D10 requires.

### Decision: a seat Codex cannot start is not compared

Every item keeps its compared key; only single-engine heddle targets and settings are keyed to one engine, so there is no exemption table.
A spec that requires forks, such as Merriam's, cannot start on Codex, whose engine meets no forks requirement, so `Start` refuses it there.
The parity tests log such a seat with its unmet requirement and compare nothing for it, and `lym shuttle manifest` refuses it naming the requirement, both through `shuttleparity.Unmet`, the engine's own requirement answer.

### Decision: `codex-engine-trial`'s Rules land on `codexengine`

The board's project `codexengine` (P1) says the project `codex-engine-trial` was merged into it on 2026-11-16, and no entry `codex-engine-trial` exists, so D11's rule goes under `codexengine`'s `## Rules`.
The decision record carries this call as the parent's `Added after Discussion` entry of 2026-11-17.
The decision record's Goal already moves D11's osl section to `codex-writer-bench`.

### Decision: seats come from the table builders

`seatengine` holds no tables; production functions build them.
The catalog holds one entry per builder with the functions holding its literals, the two unexported builders gain exported entry points in their packages, and the scan fails on a non-empty `seatengine.Table` literal whose function no entry names.
A builder runs on the current worktree's configuration, with stand-ins only for a run's own inputs, written under a directory the caller removes.
The `MultiLLM` recipe row's seats come from an operator-written row config, not code, so the catalog does not enumerate them.

## PATTERN changes

- `PATTERN-engine-parity` — adds: Launching a role on an engine: the session gets every skill, standing passage, instruction file, guard, hook and directive a session of that role gets on another engine, through its own channel; each engine's launch manifest is compared by test. (test)
