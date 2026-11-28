---
verdict: BLOCKING
findings:
  - id: F8
    severity: BLOCKING
    class: design
    location: internal/beamcli/dobbyautonomystamps_test.go:18
    summary: TestRunAutonomyStamps_PostsARefusalOncePerEpisode does not open with tiergate.Require naming its tier, as PATTERN-test-speed requires, and cmd/lym's TestTierTags_OneTierPerTestFile fails on it.
    fix: "In internal/beamcli/dobbyautonomystamps_test.go, make `tiergate.Require(t, 0)` the first statement of TestRunAutonomyStamps_PostsARefusalOncePerEpisode, before `t.Parallel()`, and add `\"github.com/Knatte18/loomyard/internal/testkit/tiergate\"` to the file's imports. Tier 0 fits: the test drives runAutonomyStamps in memory with no hub, git or tmux. Check with `lym gate test ./cmd/lym --tier 0 -- -run TestTierTags_OneTierPerTestFile`."
  - id: F9
    severity: LOW
    class: decision
    carries: F7
    location: internal/orchcli/notify.go:98
    summary: The halt notices loom and batten post through NotifyParentHalt and NotifyRunFromHalt stamp through hubgeom.Autonomy and drop the refusal. A refused value reaches the orch only when the dobby builds one of its own stamped mails, so a run under beam.run_watch off whose halts come only from loom or batten is stamped false, and the orch never learns why.
    fix: "The parent chooses between two options. Accept the gap: D1 names the dobby as the poster, and the halt is still relayed under false. Then add one sentence to the runAutonomyStamps doc in internal/beamcli/dobbydeps.go saying that a refusal is told only once the dobby stamps one of its own mails about the run. Or build the missing part: have the dobby notice a refused value on its own tick for each listed run, whether or not it posts a mail, and post through the same runAutonomyStamps episode."
prior:
  - id: F1
    status: resolved
    answer: tellRefusal builds the line and the post warning with autonomyName(), which renders beam.autonomy through configengine.KeyName, and refusalPrefix opens with the new wording.
  - id: F2
    status: resolved
    answer: "The section now holds Door one-way and Blast radius config, and its prose names the beam config load an older build or a revert refuses, the loads that run it, and the condition under which the door stays two-way. The fixer's reasons for its differences hold: the summary's parser takes one word as the blast radius, and no loomcli call reaches dobbyengine.LoadConfig."
  - id: F3
    status: resolved
    answer: "The refused row now goes on to write false, post a watch line with no refusal, write maybe again and expect the refusal mail a second time, all on the same deps. Posting through RunWatch.Post avoids the run notice's topic tag, which would supersede the unread notice. The direct tellRefusal test pins the failed-post retry."
  - id: F4
    status: resolved
    answer: "The loomcli table has an unattended row for runAutonomy, TestPawlDeps_AutonomousReadsUnattendedAsOn drives the pawl seam, and TestWireProduction_OrchAutonomousReadsUnattendedAsOn drives the Slack wiring. Each would fail if its reader compared against AutonomyOn alone."
  - id: F5
    status: resolved
    answer: "AutonomyWithRefusal returns read, and tellRefusal ends the episode only when read is true. A failure with no refusal leaves the episode as it is, and the doc comments say so."
  - id: F6
    status: resolved
    answer: tellRefusal marks the episode under the lock, posts unlocked, and clears the mark only if it is still its own after a failed post. A second concurrent first refusal sees the mark and does not post, and the doc states the ordering that results.
  - id: F7
    status: open
    answer: No added entry settles this held finding, so it is carried as F9.
---

### [BLOCKING] The new tier-0 test does not name its tier

**Location:** `internal/beamcli/dobbyautonomystamps_test.go:18`

**Issue:** PATTERN-test-speed requires every `Test` function to open with `tiergate.Require(t, N)`.
`TestRunAutonomyStamps_PostsARefusalOncePerEpisode`, added by the round-1 fix commit `ce6dee700`, opens with `t.Parallel()` instead.
Running `lym gate test ./cmd/lym --tier 0 -- -run TestTierTags_OneTierPerTestFile` on this tree fails with
`internal/beamcli/dobbyautonomystamps_test.go: TestRunAutonomyStamps_PostsARefusalOncePerEpisode does not open with tiergate.Require(t, N) naming its tier`.
Without the call, `lym gate test` never puts `internal/beamcli` in tier 0 for this test, so the only test of the failed-post retry runs in no gate tier.
The rubric's `Do not flag` list leaves a failing test to the verify gate.
The constraints section makes every violation of a listed constraint a BLOCKING finding and says the constraint wins over a conflicting rule, so this finding is raised and the conflict is named here.

### [LOW] A refused value reaches the orch only through the dobby's own mails

**Location:** `internal/orchcli/notify.go:98`

**Issue:** Carried from F7, which the parent holds.
`WithRunAutonomy`, which `NotifyParentHalt` and `NotifyRunFromHalt` use, stamps through `hubgeom.Autonomy` and drops the refusal that `AutonomyWithRefusal` would return.
The callers in `internal/loomcli/drivewiring.go`, `internal/loomcli/prepush.go` and `internal/battencli/notices.go` post a halt stamped `autonomy: false` and nothing more.
`runAutonomyStamps` tells the orch of a refusal only when the dobby builds a gate-hold notice, a run notice or a watch line.
Under `beam.run_watch` off, `internal/dobbyengine/runwatch.go:201` mails no watch line.
A run whose halts come only from loom or batten therefore gets its halts relayed under false, and the orch never learns that the value was refused.
The `NotifyParentHalt` doc says the orch handles the notice "under the run's value", which a refused value does not bear out.
No added entry in the decision record settles this.
