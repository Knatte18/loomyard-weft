# Card 2 — the build step's summary and timeout lines, run record and landing failure record

**Create:**
- `internal/gateslot#BuildSummaryLine`
- `internal/gateslot#BuildTimeoutLine`
- `internal/gateslot#buildSummaryPattern`

**Edit:**
- `internal/gateslot#RunRecord`
- `internal/gateslot#WriteRunRecord`
- `internal/gateslot#FailedTier` -> `func FailedTier(log string) (tier int, build bool, ok bool)`
- `internal/gateslot#FindTierTimeout`
- `internal/gateslot#testFailedTier`
- `internal/gateslot#TestWriteRunRecord`
- `internal/gateslot#TestWatchdogLine`
- `internal/verifytree#LandingFailure`
- `internal/verifytree#TestVerify_Scenario`
- `internal/landingshed#verifyGate.recordFailure`
- `internal/landingshed#verifyGate.verdict`
- `internal/landingshed#TestMergeIn_PlanVerifyFailureRecord`
- `internal/landingshed#TestVerifyGate_CheckMapsVerifyResult`
- `internal/gateslot/doc.go`
- `internal/landingshed/doc.go`
- `contracts/stencils/landing/landing-template-fixer.md`

**Extends:**
- `internal/verifytree/doc.go`

**Uses:**
- `internal/landingshed#writeVerifyLog`
- `internal/landingshed#newMergeInFixture`

**Intent:**
Give the build step of Decision 6 its own lines and records before `gatecli` runs it, so a verify site and a landing row can tell a compile break from a test failure.

- `RunRecord` gains `Build bool` (`json:"build,omitempty"`), set on the build step's record; its `Tier` is the first opted-in tier the build ran before, and its `ScopeForm` is `build`.
  `RunRecord` also gains `BlockedBy string` (`json:"blocked_by,omitempty"`), the log path of the failed build that kept the tier from running, which card 12 sets on each blocked tier's record.
  `WriteRunRecord` names a build record `<stamp>-build-<pid>.json` with its `.log`, in place of `tier<N>`.
- `BuildSummaryLine` renders `gate test: build before tier<N> wall <s>s exit <code> log <path>`, which no `tierSummaryPattern` reader matches, so a build line never reads as a tier line.
  `buildSummaryPattern` matches it.
- `BuildTimeoutLine` renders `lyx gate test: build timeout: build exceeded timeout_min <timeout>; ...`, naming `lyx config gate --set 'build={...}'`, carrying every key of the build and the raised `timeout_min`, from the prime, as the way forward, with the closing role clause.
  `FindTierTimeout` also finds it and returns `build` as the tier, so `verifytree`'s `tierTimeoutDetail` reads it unchanged as `gate build exceeded timeout_min <timeout>` and `Verify` marks the result timed out.
- `FailedTier` returns the first summary line in the log whose exit is not 0, tier or build line alike, with `build` true for a build line and `tier` that line's tier.
  A build that failed blocks the opted-in tiers after it without a failed `tier<N>` line, so a log whose first failing line is the build's yields `build` true.
- `LandingFailure` gains `Build bool` (`yaml:"build,omitempty"`), so an older record reads as before.
  `recordFailure` sets `Build` from `FailedTier`, takes the tier it names, and leaves `Tests` empty for a build failure, without calling the `failingTests` seam, so the landing fixer is pointed at a compile break; the trace fields are written as before.
- `verifyGate.verdict`'s timed-out reason names `lyx config gate --set 'build={...}'`, carrying every key of the build and the raised `timeout_min`, in place of the `tiers.tier<N>={...}` rewrite when `result.Detail` names the gate `build`, so a build timeout at Merge-In or Finalize names the key that raises it, and its first step reads `fix the hanging build, such as a stalled restore or compile, on the task branch` in place of `fix the hanging test on the task branch`, since a build timeout ran no test; a tier timeout keeps today's text.
- `landing-template-fixer.md`'s mode section gains one line: when the landing failure record holds `build: true`, the gate failed in the build step before any test ran, so in either mode the fixer fixes the compile error the record's log names, in place of failing tests.
- The package docs name the build line, the build timeout line, the record name, the run record's `blocked_by` field and the `build` field of the landing failure record, and `gateslot/doc.go`'s run-record name and `FailedTier` sentences and `landingshed/doc.go`'s failing-tier sentence are rewritten to cover the build line.

Tests:
`testFailedTier` gains rows: a log whose first non-zero line is a build line (tier 1, build true), a log with a passing build line then a failing tier line (that tier, build false), and a log with a failing build line after a failing tier line (the tier, build false).
`TestWriteRunRecord` gains a row writing a `Build` record and asserting the `-build-` stem and the round-trip of `Build`, and a row writing a tier record with `BlockedBy` set, asserting its round-trip.
`TestWatchdogLine`'s `FindTierTimeout` table gains the build timeout line, returning `build`, and asserts the line names `build={...}`.
`TestVerify_Scenario` gains a row beside the tier timeout row: a command printing `BuildTimeoutLine(5*time.Minute)` and exiting 1 fails as timed out with `Detail` naming `build` and `5m0s`.
`TestMergeIn_PlanVerifyFailureRecord` gains a row whose verify log, written by `writeVerifyLog`, holds a failing build line before tier1, with `fx.m.gate.recorder.failingTests` set to return one test: the record has `Build` true, `Tier` 1 and no `Tests`, so the row fails if the seam's answer leaks into a build failure.
`TestVerifyGate_CheckMapsVerifyResult` gains a timed-out row whose `Detail` names `gate build exceeded timeout_min 5m0s`, asserting the exact reason opens its way forward with `fix the hanging build, such as a stalled restore or compile, on the task branch` and names `build={...}`, not `tiers.tier<N>` nor the hanging test; the existing tier2 timed-out row keeps its exact text.

**ImpactSummary:** `FailedTier`'s one production caller, `recordFailure`, is re-signed here, and the landing timed-out reason changes only for a build timeout; the new fields are omitempty, so every existing record and log reads as before.
