```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix's change set for agreement with sibling decisions, criteria and cross-references (fix-consistency lens).

- **MEDIUM**: D9's walk bullet ("A merge commit fabric wrote by D5's strip passes it when its tree, records paths stripped, equals the clean merge of its two parents' records-stripped trees") defines a fabric merge differently from D7's rewritten predicate ("carrying fabric's merge trailer whose tree equals its first parent's records paths over the clean merge of its two parents' records-stripped trees (D5)").
  - The D9 test drops the trailer and the check that the records equal the first parent's.
  - So the record now has two definitions of "a merge commit fabric wrote", and they admit different commits.
  - Evidence: D5 says D7's records-only test and D9's code identity "read them the same way", and the D9 bullet next to it already points to D7 for records-only commits ("as D7 defines them"). The merge clause gets no such pointer.
- **LOW**: the new `PATTERN-run-fsm` entry under PATTERN changes covers only "a prime records push origin refuses as a protected branch".
  - D7 also records the guard's own refusal as `protected` once a protected record exists for the prime's branch ("Once such a record exists for the prime's branch, the guard's refusal is recorded as `protected` too").
  - Under the new seam rule, that transition also completes with its records unpushed, so the declared exception is narrower than the behaviour D7 decides.
- **LOW**: the Constraints bullet that lists the PATTERN backgrounds the Webster card extends (`PATTERN-fabric-boundary`, `PATTERN-lym-files`, `PATTERN-cwd-resolution`, `PATTERN-pair-remote`) does not name `PATTERN-run-fsm`.
  - The fix added a `PATTERN-run-fsm` change, and that pattern's background section "An owed push holds the run" names only transport and teardown-lock pushes as owed.
  - That section goes stale now that `operator_unpushed` is owed and `protected` lets the run act on.
  - Evidence: `pattern/PATTERN-run-fsm.md:29-34`, as the earlier grep showed.

Further tool calls: none.
