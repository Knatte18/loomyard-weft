## Goal

```markdown
## Seen (research, landed 2026-11-01)

`docs/research/gate-fewer-binaries-trial-2026-11-01/report.md` measured three ways to cut the new test binaries a gate run builds; on OSL-1046 the virus scanner holds each new binary about 10 s, several in parallel, so a worst-case gate run there takes 12 to 13 minutes against 2 to 3 on Neve (operator, 2026-11-01).
Its verdict: one test binary per package for all tiers, the tier chosen at run time, is the cut worth making permanent.
On the module's package graph it takes a cold run from 532 test binaries to 182, and a change of mean fan-in from about 101 new binaries to about 35; on a ten-package sample every test that passed before still passed.

## Design (operator, 2026-11-01)

- The test tier is chosen at run time by `LYM_TEST_TIER`, not by build tags, so each package builds one test binary that holds every tier's tests.
- `LYM_TEST_TIER` names one tier (`1`) or a range up to a tier (`0-2`).
- Every Test function calls the tier's `Require`, which skips it when its tier is not among those named; the report's trial 2 shows the form.
- `gate.yaml` gives each tier an `env` entry setting `LYM_TEST_TIER`; the tag guards, `go vet` per tag and the gate's `tier` command follow.
  A `--upto` run may pass a range and run several tiers in one `go test` call per package where their gate slots and timeouts allow; tier2's own slots and timeouts stay as they are.
- `PATTERN-test-speed` changes from disjoint tags to the run-time selector.
- Known costs the report names: tier3 dependencies (`llmkit`) join every binary and raise its fan-in, a binary is larger, and a test that reads `LYM_TEST_TIER` loses its test-cache entry per tier.

## Shape of the run

- The tag removal and the `Require` calls across about 180 packages are made by one script over the whole module, as trial 2's `untag.sh` did, never by hand.
- When the gate then fails in many packages, the darn writer groups the failing packages into batches of a fitting size and hands each batch to a Sonnet subagent of its own, then gates the whole module once.
- Like a rename it runs with no other code-changing run in flight; a docs-only research run beside it is fine.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-01): The tier falls back to the build tag until the hub gate config names it

Decision: The hub's gate.yaml is not changed during this run: lym config --set cannot write a key inside a tier's env map, and the editor needs an interactive terminal. The run-time selector therefore reads LYM_TEST_TIER when it is set and otherwise falls back to the tier of the build tag the binary was built with, through one small non-test file per tier tag in the selector package (for example a //go:build tier1 file setting the default to 1). With the hub's current commands, -tags tierN, every tier call then runs its own tier's tests. Once the run has landed, the orch has gate.yaml give each tier env LYM_TEST_TIER and drops -tags and the per-tag go vet calls, which is when one binary per package takes effect; the change description says so.

Rationale: Keeps every worktree's gate correct without a hub config edit that lym cannot make non-interactively today.

### Added after Discussion (parent, 2026-11-01): A Go rewrite program makes the mechanical change

Decision: The writer may write a Go program under .scratch/ that drops the tier term from the test files' //go:build lines and opens each Test function with the Require call, and run it once with go run over the module's test files; it is not committed. Everything else changes through Edit and Write.

Rationale: The record asks for one script over about 1645 test files; the repository's convention for mechanical rewrites is a Go program, and the shell-edit guard does not refuse go run.

### Added after Discussion (parent, 2026-11-01): No Sonnet subagents in this session

Decision: The darn session has no Agent tool, so the writer fixes failing packages itself instead of handing batches to Sonnet subagents; it batches the fixes per gate report as the session's instructions say.

Rationale: The record's subagent batching cannot run in this session.

### Added after Discussion (parent, 2026-11-01): The hub gate config names the tier now; no tag fallback

Decision: Superseding the tag-fallback decision: the orch has set env LYM_TEST_TIER (0, 1, 2, 3) on each tier in the hub's gate.yaml, beside GOFLAGS, keeping the commands' -tags tierN. The selector reads LYM_TEST_TIER only; no per-tag fallback file is written. After landing the orch drops -tags from the commands and the per-tag go vet calls.

Rationale: The operator had the orch edit gate.yaml directly; main ignores the variable, so nothing changes for other worktrees.

### Added after Discussion (parent, 2026-11-01): Sonnet subagents for failing package batches after all

Decision: Superseding the no-subagents decision: the writer's session does offer the Agent tool (its tool list carries it). When the gate fails in many packages, the writer groups the failing packages into batches of a fitting size and hands each batch to an Agent subagent with model sonnet, all launched in one message, each told the packages it owns, the gate rule from the writer's own system prompt with its timeout, and to edit only its own packages; the writer then gates the whole module once. A handful of failures the writer fixes itself.

Rationale: The operator asked for Sonnet subagents; the earlier decision rested on the writer's belief that it had no Agent tool, which its transcript contradicts.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
