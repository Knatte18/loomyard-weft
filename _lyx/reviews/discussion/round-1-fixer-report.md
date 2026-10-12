# Fixer report — discussion, round 1

## Changes

- F1 (BLOCKING), fixed as proposed, with one widening.
  `decision-record.md` D2 now rejects a dollar expansion in a reader segment: in the single-quote-blanked view, a `$` followed by `{`, `[`, a name character or a special-parameter character.
  D2 names the command-level guards that bound reader segments (substitution markers, output redirections, backgrounding), states that `<` input redirection stays admitted, and says the `IsReadOnlyCommand` doc comment's expansion sentence becomes true for readers.
  Goal item 2, the Scope In bullet, the Scope Out bullet, D2's heading and Bound, and the Acceptance criteria row (`cat ${x@P}`, `head -n ${a[y]} f`, `cat $x` false; `grep 'a$' f`, `grep "^a$" f` true) follow.
  Widening: the rule also rejects the special-parameter characters (`@`, `*`, `#`, `?`, `$`, `!`, `-`) after `$`, which the proposed name-character rule left open.
  `$@` and `$*` expand under `GLOB_SUBST` the same way `$x` does, and a regex anchor never puts one of these right after `$`, so the widening costs no everyday read.
- F2 (MEDIUM), fixed as proposed: D1 Bound lists the run's plan among the gated figures, says why the flag can move the plan selection to an earlier run's plan or none, and the flag's doc-comment description says the plan, peak table, start-table base and identity card counts may come from an earlier run, or the run may have no plan commit before its first fork.
- F3 (LOW), fixed as proposed: D1 Bound says a transcript with no timestamped user or assistant line is counted under the flag, and the flag's description replaces the doc comment's "left out when a bound exists" sentence.
- F4 (NIT), fixed as proposed: Q7 moved below Q6 in `support-log.md`'s Question ledger.

## Disputed

None.

## Deferred

None.
