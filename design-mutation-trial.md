# Run the mutation trial with lym gate mutate

`mutation-trial` · Task · research · starting · Open-IDE

Run the mutation trial the board entry asks for, with `lym gate mutate`, then decide whether the `mutation` report step is built.

## The trial

Replay exactly 20 landed diffs of the repository with `lym gate mutate`, one run per diff and no repeats, and count and read the survivors.
Each replay needs a worktree at the landing commit, which no implementer of the run that built the verb may create.
In it, run `lym gate mutate --base <the landing's first parent>`.
The verb refreshes a task worktree's own maps of the changed packages itself, so a worktree with a clean tree needs no map build first.
In the prime it judges only the packages whose shared maps match the worktree, which `lym gate test --map` builds.

Pick landings that change Go production code, across several packages, small and large.
Per diff, record the counts the envelope prints: mutants run, killed, survived and unjudged, the uncovered changed blocks and the packages not judged.
Read a sample of the survivors and judge each: a missing check, or a mutant no test could reasonably kill.
Read the unjudged mutants and the packages not judged too, since a high share of either says the verb's bounds need tuning before it can pay.

## The decision

From the counts, decide whether the `mutation` gate report step is built, as a later darn, and listed in the template's `diff` keys.
The step would write `mutation.json`, print one summary line and feed a `mutation_report` marker of the code review rubric.
It is built only if the survivors the trial reads are mostly findings a reviewer would act on, at a cost in build and run time that the gate's bounds accept.
