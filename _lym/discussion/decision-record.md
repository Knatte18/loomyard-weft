## Goal

Measure what one gate run builds and launches, per tier, and rank the candidate cuts by the new binaries each saves.
The report lands in `docs/research/`; the cut itself is a later task.

## Scope

- In: one research report under `docs/research/gate-fewer-launches-2026-10-31/report.md` (a dated directory, as the other research entries), holding the count of new binaries and launches per tier, the candidate cuts ranked by new binaries saved, and the gain on OSL-1046 computed at about 10 s per new binary.
- In: two gate runs on Neve, each measured once: one `lym gate test --upto 2` and one over the whole suite.
- Out: any code, config or gate.yaml change; the cut; a run on OSL-1046.

## Decisions

### D1 — Report only, no code change

Decision: the commit adds the report and nothing else; no gate code, tier command or config is touched.
Rationale: the board entry says this task measures only.

### D2 — Counts come from the gate's own records plus a static derivation

Decision: the per-tier count of new binaries is derived from the gate run's tier records and logs (`lym gate report`, the tier log under the gate directory) together with `go list` over each tier's tag set, which gives the packages that link a test binary per tier; the report states which numbers were observed and which derived.
Rationale: the cap is two gate runs, so a count that needs a third run is derived from the package graph instead; the code and tiers are the same on Neve and OSL-1046, so the count transfers.

### D3 — The two runs are a small explicit scope and the whole suite

Decision: the `--upto 2` run takes an explicit small package set as targets, since the clean branch has an empty diff and a diff scope would run no tier; the second run is `--upto 2 --all`.
The report names both command lines.
Rationale: an empty-diff `--upto 2` measures nothing, and the two points (small and whole) show which launches scale with packages and which are fixed per run.
Bound: this reads "one `--upto 2` and one of the whole suite" as two scopes of the same tier range; a third run is never made.

### D4 — A new binary is one with a hash the host scanner has not seen

Decision: the count separates new binaries (test binaries and `go vet` or build outputs whose content changes from run to run or from key to key, and the prebuilt `lym`) from launches of known binaries (`go`, `vet`, `compile`, `link`, `git`, `tmux`), and prices only the first at about 10 s each.
Rationale: the operator's cause statement is that Cortex XDR holds back an unseen hash; a known binary start is cheap.

### D5 — Candidate cuts ranked by new binaries saved

Decision: each candidate (fewer test binaries by merging packages, one test binary reused across the gate keys of a run, one shared prebuilt `lym`, fewer `go vet` link steps across the four tags, test-spawned `go build` or `lym` builds) is listed with the binaries it saves per run and the saving on OSL-1046, ranked by that saving, with the risk to result equality stated.
Rationale: the follow-up task picks from the ranking.

## Constraints

- No code change; the only added file is the report (an inert path, no test reads it).
- Gate runs go through `lym gate test --wait`, never a raw `go test`.
- The PATTERN `PATTERN-gate-slot` and `PATTERN-test-speed` apply to how the runs are made.

## Auto-mode assumptions

- The board entry's "whole suite" means every tier but the manual one, `--upto 2 --all`.
- `launch_seconds` in `lym gate report` is read as given, without re-deriving it.

## Open risks

- Neve may hold a warm build cache, so its count of rebuilt binaries is a lower bound for a cold OSL-1046; the report states the cache state of each run.
- A test binary's hash may repeat across keys through the cache even when the build is logically new; the report counts hashes where it can observe them and says where it cannot.

## Acceptance criteria

- D1: the diff touches only files under `docs/research/`.
- D2: the report separates observed numbers from derived ones, and names the source of each.
- D3: the report names both command lines and the scope each measured.
- D4: the report gives a count of new binaries and a count of known-binary launches per tier, and the OSL-1046 gain at about 10 s per new binary.
- D5: the report ranks the candidate cuts by new binaries saved, each with its result-equality risk.
