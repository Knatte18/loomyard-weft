```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds, find the `TestStartGuard_Refuse` row (or other test) that would fail if that behaviour broke.

**Behaviour → test that pins it**

1. **A grouping project's run on record with the filing mark standing passes to the task checks.** Pinned by `FiledRunUnderItsFamilysProjectResumes` (startguard_test.go:294). Without the change, `refuseProject` returns the grouping-project refusal and the row's want-nil fails.
2. **A grouping project's run on record with no filing mark is still refused.** Pinned by `UnfiledRunOfAGroupingProjectIsRefused` (startguard_test.go:305). A guard that passed every found run of a project would fail this row. It is not a duplicate of `GroupingProjectRefusedNamingTheCreate` (:281): that row has found=false and never reaches the `FilingMarked` call.
3. **The `found &amp;&amp;` condition: a filing mark with no run on record is still refused.** Pinned by `GroupingProjectWithAFilingMarkButNoRunIsRefused` (startguard_test.go:288). Dropping `found` from the condition fails this row.
4. **A failed mark read refuses through `probeRefusal`, judged=true.** Pinned by `FilingMarkProbeErrorRefusesNamingTheProbeAndTheRerun` (startguard_test.go:313). Two breaks fail it, each on the substrings it asserts: the error being swallowed, or falling through to the grouping refusal.
5. **The hub wiring `FilingMarked: shedrun.FilingMarked(location, slug)` (startguard.go:125).** No test pins it. Each of these breaks passes every test: removing the line (nil seam, so every filed run is refused again), reading another run ID, or reading another location. Only the fake seam in `TestStartGuard_Refuse` exercises the field. `beamland_integration_test.go` drives `hubStartGuard` against a forged hub and seeds a non-beam project `g` (line 60). No row there writes a filing mark with `shedrun.WriteFilingMark` and then resumes the run under the guard. This wiring is the production path of the fix: the guard must read the mark that raddle's `MarkFiled` writes, in the prime and under the slug (raddlewiring.go:264-270).
6. **A nil `FilingMarked` reads every run as one that filed nothing (doc contract, startguard.go:55).** No row sets `Project` and leaves `FilingMarked` nil. `NilSeamsCheckNothing` (:363) leaves every seam nil, so `refuseProject` never runs.

**Findings**

- **MEDIUM**, class `design`. `internal/battencli/startguard.go:125`. No test pins the hub wiring of the filing-mark seam. Removing the line or reading the wrong run ID or location passes the whole suite, and the run would be stuck at Run-Shed again.
  Evidence: `internal/battencli/startguard_test.go:446` is the only place the seam is set, and it is a fake. `internal/battencli/beamland_integration_test.go:60-130` drives `hubStartGuard` but never writes a filing mark. A grep of battencli and beamcli tests for `WriteFilingMark` finds it only in cleanup_test.go and commitstatus_test.go.
  Fix: add a subtest to the hub-forged guard test that writes `shedrun.WriteFilingMark(h.Location, "g")` for the grouping project `g` and asserts that `hubStartGuard(h.Location, "g").refuse("g", "run", runShed, true, true)` passes. A sibling subtest without the mark (or on another slug) asserts the grouping refusal.
- **NIT**, class `design`. `internal/battencli/startguard.go:55`. No row pins the nil-`FilingMarked` contract while `Project` is set. Only hand-populated receivers in tests reach it, so a row is optional.
  Evidence: `internal/battencli/startguard_test.go:363-367`, where `NilSeamsCheckNothing` sets every seam nil.

Outside this lens: in `FiledRunUnderItsFamilysProjectResumes`, the `dirty` and `parts` fields pin nothing that is specific to the filed path. With found=true and producer Run-Shed, the task checks skip the dirty-source and left-pair probes for any resume. The summary's claim "dirty source and left pair parts included" is therefore weaker than it reads.

Tool calls:
1. Grepped the battencli and beamcli tests for `WriteFilingMark`/`FilingMark` and read `beamland_integration_test.go:60-130`. My context held no test that drives `hubStartGuard` with a filing mark on disk, and I needed to know whether one exists.
