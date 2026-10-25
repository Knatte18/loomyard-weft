```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-3 change set agrees with the sibling text it touches, and find any twin left unchanged.

- **MEDIUM**: D10's new origin-reconcile paragraph ("it leaves that weft unmoved and reports the branch stopped … never fails its caller for it") contradicts D2.
  D2 (line 50) says "Every `fabricengine` path that acts on a marker, D5 to D10, refuses an own marker that is present but empty, naming the file and the way forward, before touching anything."
  The origin reconcile now acts on a marker in D10, so D2 has it refuse.
  D10 says it only stops and reports, and it never says what an empty marker does there.
  Evidence: D2 line 50 against the new D10 paragraph.
- **LOW**: the empty-marker acceptance criterion was left as it was after the origin reconcile became a marker-comparing path.
  "An empty own marker is refused by checkout, pull, push, `Add` from it, the anchor verb and reconcile, each touching nothing" names `lyx fabric reconcile` (D8), not `ReconcileOrigin`.
  Its twin is the D5 list, which the fix extended to "D10's pull, push and origin reconcile comparison".
- **LOW**: D6 got no twin of the F4 change to D7 step 1, which reads "every run, a re-run included, writes its own pid and start time into the record".
  D6's `Add --anchor` "takes D7's move lock and writes its move record", but nothing says it writes a pid.
  So the move-lock refusal of a `--set` raced against a create has no stated pid source.
  The phrase "every run" reads as runs of the verb.
- **LOW**: the landed-recovery bullet still says "runs steps 1 to 3 and 5 to 8 idempotently, its step 1 keeping the record", while step 1 now has every run, a re-run included, write its pid and start time into the record.
  "Keeping" should say it keeps `A`, `B`, the kind and `U` while rewriting the pid.
  The move-lock refusal bullet ("the refusal names its pid, read from the move record") also gives no "starting" case, which only the new step-1 sentence states.
  The two places should agree on what the refusal names.

No further tool calls.
