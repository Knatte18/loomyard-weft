```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens (callers, tests, docs and rules that the diff forces outside its own lines).

**Per change, the set found:**

1. **`shuttleengine.ReadNewestRunEnd` (new function)**
   - The only caller is `readNewestAgentEnd` in `internal/battencli/wire.go`.
   - `internal/shuttleengine/doc.go` and the `activity.go` header both mention it. Covered.
   - It reuses `unreadableFiles`, so it shares that error-episode state with `ReadAgentActivity`. Consistent.
   - No PATTERN entry is touched. The `pattern/` background files and `PATTERN.md` are not in the diff, and I found no declared PATTERN change in the directive. Covered.

2. **`RunNoticeDeps.NewestAgentEnd` (new seam field)**
   - The constructors found are `battencli.DobbyRunNotices` (wired) and the `newNoticeHarness` test fake (wired).
   - The nil case is handled as before: `noticeCondition` checks `seams.NewestAgentEnd != nil`.
   - The beam and prime dobby deps are both built through `DobbyRunNotices`, so both get the seam. Covered.
   - `internal/beamcli/dobbydeps_integration_test.go:75` asserts that "every seam" of both dobbies is filled. Its condition lists `ReadChild`, `DriverAlive`, `Agents`, `Pause`, `ReviveAPIError` and `MarkerPath`, but not `NewestAgentEnd`.
     - That is a seam-wiring assertion the diff left stale: the new seam is the only one a dropped wire would leave nil unnoticed.
     - Finding F1 below.

3. **`judgeAgents` signature (new `newestEnd` parameter)**
   - The only caller is `noticeCondition`, and no test calls it directly. Covered.
   - `doc.go` line 138 describes the new fallback, and the function's own doc comment is updated. Covered.
   - Nothing else describes the old "later of history and since" wording. The search for the notice text and "newest history entry" finds only `doc.go` and the tests.

4. **Notice text and episode stamp**
   - The text "no agent activity readable for …" is unchanged, and the stamp is still the newest time. Covered.
   - The tests that assert the text are updated or added in `runnotice_test.go`.

5. **New episode key `|agent-end` in `noticeFailures`**
   - It is the same pattern as the existing keys. No other reader.

6. **Stencils, contracts and specs**
   - The search of `internal`, `contracts`, `docs` and `PATTERN.md` found no stencil or spec describing the no-run fallback. Stencil wording is not touched.

7. **Commit message**
   - The message says `ReadNewestRunEnd` "reads the newest ended run's record time" and that the new seam feeds the fallback. Both match the diff.
   - No `Pattern-Departure:` trailer is present, and none is needed.

8. **`wire_seams_test.go`**
   - Updated for the absent-worktree case of the new seam. Covered.

**Findings**

- **F1, LOW, consistency** — `internal/beamcli/dobbydeps_integration_test.go:75`. The "every seam filled" check omits `notices.NewestAgentEnd == nil`, so a dobby built without the new seam would pass it. Evidence: the condition at line 75 lists six seams and not `NewestAgentEnd`. Fix: add `|| notices.NewestAgentEnd == nil` to that condition.

**Tool calls**
1. grep for `RunNoticeDeps{`, `judgeAgents(`, the notice text and the doc phrase across `internal`, `contracts`, `docs`, `PATTERN.md` and `pattern`: my context did not hold the other constructors, callers or doc mentions.
2. Read `internal/beamcli/dobbydeps_integration_test.go` lines 65–95: to see the seam-filled assertion that grep surfaced.
