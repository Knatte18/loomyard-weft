```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked every sentence the fix added or rewrote against the code at HEAD.

Two findings. Every other claim holds.

**MEDIUM — card 20, Intent, the sentence ending "`stepCommandRows_FollowTheTasksLocalSwitch` sets `--open-terminal` in place of `Forced` and keeps its marker assertions, which prove the forced Open-Window run through the entry gate".**
Summary: the test cannot keep its off-case marker assertions unchanged under the entry gate, and setting the flag alone does not force the answer.
Evidence:
- The test drives every row through `stepCommandRow` (`internal/battencli/lifecycle_integration_test.go:1192-1203`). That helper calls `t.Fatalf` unless `step.Producer == row && step.Outcome == shedengine.Done`.
- Under card 17 a gated-off row records `OutcomeSkipped` and moves on to its `OnDone` row in the same step. The off-case steps at lines 1268 and 1273 (Open-IDE off, Open-Window off) would therefore fail the helper before the "want none" marker assertions run.
- `stepCommandRow` is not on card 20's Edit list.
- `c` is wired once by `wireForHub` in `newCommandRowFixture` (line ~1150). The forced answer is built in `battenCLI.wire`, so the forced half needs a re-wire with `openTerminalFlag` set. Setting the flag on the already-wired `c` changes nothing.

**LOW — card 17, Intent, the added sentence saying "the Field ownership sentence listing the Shed-owned fields gains `entered_row`".**
Summary: the sentence it joins calls those fields "rewritten on every persist", but card 17's own Intent says the resume and halt writes keep `entered_row`.
Evidence:
- `internal/shedengine/doc.go:109`: "current_producer, state, pause_reason, error, activity and history are Shed-owned and rewritten on every persist."
- Card 17: "`persistTransient` and the resume write in `stepLocked` pass false", and "A resume or a halt re-calls the entered row without evaluating it, since those writes keep the field."
- The card does not say how the joined sentence reads once a field is kept rather than rewritten.

Tool calls:
- Read `cmd/testtiming` grep hits, `internal/shedengine/doc.go:105-120` and the `TestPrebuildLyx` rows: to check the card 13 claims about `resolveTags`, `redundancyTags`, the real-git hint and the prebuild rows, and the card 17 Field ownership claim. My context showed only `main.go:155-190` and `prebuild.go`.
- Read `lifecycle_integration_test.go:1255-1300`: to check the card 20 claim about the marker assertions. My context showed only the grep hit at line 1277.
- Read `stepCommandRow` and the fixture's wiring: to check whether the test's steps go through the entry gate and how `--open-terminal` reaches `Env.Setting`.
