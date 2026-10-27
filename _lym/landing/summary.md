# Bench solo Codex Astra at high effort on P4

Adds `docs/research/codex-astra-solo-high-bench-2026-10-27/`: a report, its data and the harness for one `gpt-6-astra` review of the `P4` (`machine-handover`) plan at `model_reasoning_effort="high"`.
It answers how much effort alone changes a solo Astra plan review, set beside the medium solo run and the Claude fan of `docs/research/codex-astra-solo-bench-2026-10-25/`.
No Go code, `PATTERN.md` or `doc.go` changes.

## Result

High returned 3 findings against medium's 4, all real on reading and none false, and the fan's `BLOCKING` verdict as medium did.
It reproduced 1 of the Claude fan's 11 findings where medium reproduced 2, and none of the fan's 3 `BLOCKING` ones in either run.
It cost 487 s against 166 s, 134k fresh input tokens against 77k, and 5.6k reasoning tokens against 1.0k.
It kept the skip-variable bypass and the bundled public APIs, dropped the two prime-claim findings and added one: card 3's post-push `Lost` rule against card 8's absent-entry pass.
The report states that one input, one run and one grader cannot rank the two efforts.

## Design decisions

- The harness is copied from the earlier benches with only the root paths changed; the effort is the `run.sh` argument.
  `build.sh` and `setup-codex-home.sh` come from the fan bench, whose builder takes the code commit as an argument, and `run.sh`, `dump.sh` and `windows.sh` from the solo bench.
- The input is rebuilt from the solo bench's weft and code commits, so the prompt equals the medium run's.
- The plan reports only a weekly window, so no 5-hour points are given and the 70 percent stop was checked on the weekly reading, which stayed at 8 percent.
- The medium solo and fan rows are cited from the solo bench, not re-run, as the one-call cap requires.

## Verification

Exactly one `codex exec` call was made and it returned 0.
Every finding is graded in `data/match.tsv` against the fan's `P4` findings and the cited card lines.
The change is documentation and shell scripts under `docs/research/`; no build or test reads it.
