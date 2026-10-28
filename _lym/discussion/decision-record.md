# Decision record — codex-code-review-bench

## Goal

Measure whether the forked Codex Astra fan, run as the Code-Review reviewer, finds what the Claude Code-Review fan found on two landed darn diffs.
The deliverable is `docs/research/codex-code-review-bench-2026-10-28/report.md` with the harness and raw outputs under `data/`; no code under `internal/` changes.

## Scope

- In: choose two inputs, build the Code-Review prompts from the shipped stencils at the commit each Claude round reviewed, run 4 `codex exec` handlers (each input twice, `gpt-6-astra`, medium), grade by hand, write the report.
- In: adapt the harness of `docs/research/codex-v2-p1-bench-2026-10-27/data/harness/` into the new bench directory.
- Out: any other Codex run, high effort, reruns beyond one retry of a run that failed to start, and any change to `internal/`, stencils or recipes.

## Decisions

### D1 — Inputs are `spawn-text-file-busy` and `fixer-overcap-review`

Both are darn runs that landed on `main` on 2026-10-28 with a diff of real Go code and a round-1 Claude Code review that returned findings.
`spawn-text-file-busy` touches `internal/shedverbs`, `internal/proc` and `tools/deploy`, and its round returned `BLOCKING`.
`fixer-overcap-review` touches `internal/landingshed`, and its round returned `APPROVED` with several `MEDIUM` findings.
The two share no package, and `spawn-text-file-busy` gives a `BLOCKING` verdict to compare against.

### D2 — The reference is each run's round-1 `round-1-review.md`

It is read from the weft history of the run's Code-Bouncer commit (`_lym/reviews/code/`), with the lens files beside it for provenance.
The diff under review is the code branch at the commit that round reviewed, found from the landing commit's parentage and the round's start time.

### D3 — Lenses are the three Code-Review lenses the Claude fan ran

`code-correctness`, `code-affected-set` and `code-tests`, read from the shipped Burler stencils at the code commit the harness stages, in place of the plan lenses.

### D4 — Reading directions are block v2 with plan-only lines dropped

Each dropped line is named with its reason in the report.
The fork cap stays at 20 tool calls.

### D5 — Context parity as the plan benches ran it

The PATTERN overview is inlined as production renders it, and the snapshot's `CLAUDE.md` is loaded as `AGENTS.md`.

### D6 — Grading is by hand, by the writer, per run

Each Codex finding is matched to a reference finding, counted as real-not-in-reference, or counted as false.
Per run the report records reference findings found (and `BLOCKING` ones), unique real findings, false findings, the verdict against the fan's verdict, fresh input tokens and wall time.
The bound: one grader and one run per cell give a first spread measure, not a significance claim, and the report says so.

### D7 — Harness runs in a scratch tree, outputs copied into `data/`

Snapshots and `CODEX_HOME` directories live under `.scratch/` in this worktree and are removed by literal path at the end.
The auth link is the plan benches' (a symlink to the operator's `auth.json`).
Bound: the run reads the operator's Codex sign-in, spends only the 4 capped handler runs, and writes nothing outside the worktree.

## Constraints

- Exactly 4 `codex exec` handler runs; one retry only for a run that failed to start.
- Build nothing in `internal/`; PATTERN stays unchanged.
- Scratch files under `.scratch/`; markdown with one sentence per line.

## Auto-mode assumptions

- The Claude fan's round-1 review is the reference even where the fixer later disputed a finding.
- A finding of the reference counts as found when a Codex finding names one of its members, as the plan benches counted.

## Open risks

- A Codex handler could write outside its snapshot; the sandbox is `workspace-write` on the snapshot only.
- The rate-limit window of the shared account may stop a run; the plan benches' stop at 70 percent applies.

## Acceptance criteria

- D1: the report names both inputs, their landing commits and why they qualify.
- D2: the report lists each reference finding with id and severity, and the commit each diff was checked out at.
- D3: the report names the three lenses and each handler spawned one fork per lens.
- D4: the report names every dropped line of block v2 and the reason.
- D5: `data/context-hashes.txt` shows the `AGENTS.md` hash equal to the snapshot's `CLAUDE.md`.
- D6: the report has a per-run table with every column named in D6 and a spread section for the two runs per input.
- D7: raw outputs sit under `data/`, `.scratch/` is gone, and exactly 4 handler rollouts exist.
