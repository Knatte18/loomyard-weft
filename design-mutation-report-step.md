# Build the mutation gate report step with the trial bound changes

`mutation-report-step` · Task · feature, gate, testmap

## Source

The mutation trial (`docs/research/speed/mutation-trial-2026-11-19/report.md`) replayed 20 landed diffs with `lym gate mutate`:
430 mutants, 374 killed, 44 survived, 12 unjudged; 26 of the 44 survivors are checks a reviewer would act on, spread over 11 of the 20 diffs.
The runs took 30 s at the median and 80 s at most. Its verdict: build the step after three bound changes.

## Idea

Build the `mutation` gate report step, listed in the template's `diff` keys, with the trial's three changes:

1. Spread the mutants across the changed functions, one per function first, so `mutation_max_mutants` no longer judges a large diff on its first functions only.
2. Skip as a mutation site a statement that only logs and a branch that tests `runtime.GOOS` for another platform.
3. Write the full list to `mutation.json` and print one summary line: mutants run, killed, survived, unjudged, uncovered changed blocks, packages not judged.

The step feeds a `mutation_report` marker of the code review rubric with the survivors.
