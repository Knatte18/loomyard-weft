```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens, walking each rule the diff adds as an instruction against concrete cases.

**Per-branch list**

1. Spec `### A bug entry's record`, entry **not** labelled `bug` (labels `["enhancement"]`): the rule doesn't apply and nothing changes. Correct.
2. Spec, entry labelled `bug` with a reproducible fault and a known cause: Goal, Constraints and an AC bullet carry the fault, the cause and a test that fails on the merge base. This matches the gate's base: `lym gate red-first` defaults to the merge base with the parent branch (`internal/gatecli/redfirst.go:48,63,95`). Correct.
3. Spec, a new test that calls a function the fix adds: it fails on the base only because it does not build there, and the gate counts that as red (`internal/gatecli/redfirst.go:67`). The spec's line "The test reaches the bug through code the merge base already has" rules this out, and it agrees with `contracts/stencils/darn/darn-template-write.md:108` and the gate's green way forward (`internal/loomshed/redfirstgate.go:96`). Correct.
4. Spec waiver clause, with a record AC that says "no test seam" and no added entry: the gate parses added entries only (`internal/loomshed/redfirstgate.go:53-61`), so it still runs. The spec's statement holds. With an added entry `### Added after Discussion (parent, …): No test seam: …`, the gate passes and runs nothing (`redfirstgate_test.go:23,51`). Correct.
5. Spec waiver clause, with an added entry titled in lower case (`no test seam: …`) or recorded by the operator rather than the parent: the gate compares case-insensitively and ignores who recorded the entry (`redfirstgate.go:58`). "Opens `No test seam`" fits. "Only the parent's … entry" is narrower than the gate, which lifts on any `lym loom decision add` entry. This is harmless because the operator is a legitimate recorder too, so it is no finding.
6. Spec, a bug whose fix changes a non-Go file of a Go package, or a hub whose scoper is not go: the gate exits 5, unjudged, and only a `No test seam` entry lifts it (`internal/gateslot/redfirst.go:14-17`, `redfirstgate.go:102-105`). The spec offers only "the test that fails on the merge base … or why no test seam exists". A record that names a real failing test for such a fix satisfies the spec and the rubric, yet the gate still blocks. See finding F1.
7. Stencil paragraph, telling the writer how to detect the label: the Step 1 `lym board get` envelope is `{"task": result.Task}` (`internal/boardcli/cli.go:230`), and the task carries `labels` (`internal/boardengine/store.go:47`). `bug` can be detected there. Correct.
8. Stencil paragraph saying the fault and the cause are "facts Step 2 researches": Step 2's fence admits "a fact the design rests on" (`loom-template-discussion.md:113-117`). When research cannot settle the cause, the spec's "or names what is still unknown" covers it. There is no contradiction with the "never interview questions" clause. Correct.
9. Rubric bullet against a record that names a root cause with no evidence: the spec requires "the evidence it rests on" (`loom-discussion-spec.md:340`), but the rubric bullet asks only for "the root cause or what is still unknown about it" (`loom-rubric-discussion-review.md:453`). Such a record passes the bullet. This agreement between the rubric and the spec is the affected-set lens's to judge, so I only note it here.
10. The end-to-end flow, for a tapestry run (Discussion-Write → Plan → Webster) of a `bug` entry: the only recipe with a `red-first` row is `contracts/recipes/darn-recipe.yaml:140`, so for this path "what the red-first gate later checks" never fires. The gate reaches the record only when a darn Record-Write reads a discussed entry's stored record (`internal/loomshed/recordwrite.go:26,177`). This framing comes from the answer key's Goal, so it is no finding.

**Findings**

- **F1.** Severity LOW, class `design`. Location: `contracts/specs/loom-discussion-spec.md:341-342`.
  The bug-record rule's escape clause names only "no test seam". It leaves out the gate's unjudged case: a fix that changes a non-Go file of a package, or a hub whose scoper is not go. The gate then fails with exit 5 however good the named test is, and only a `No test seam` entry lifts it.
  A record that follows the spec therefore does not "state up front what the red-first gate later checks" for such a fix. The darn stencil names both cases: "When the bug has no seam a test can reach, or the check cannot judge it".
  Evidence: `internal/gateslot/redfirst.go:14-17`; `internal/loomshed/redfirstgate.go:102-105`; `contracts/stencils/darn/darn-template-write.md:110-113`.
  Fix: make the AC alternative read "or states why no test seam exists or the check cannot judge the test", and mirror that in the rubric bullet and the stencil paragraph.

Tool calls:
- `sed`/`grep` on `internal/boardengine/view.go` and `internal/boardcli/cli.go`: to confirm that the `lym board get` envelope carries the task's `labels`. My context had the `Labels` JSON tags but not which struct the get verb prints.
