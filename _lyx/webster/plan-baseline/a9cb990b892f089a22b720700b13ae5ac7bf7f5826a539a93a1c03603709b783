# Card 23 — shuttleengine-fixture

**Create:**
- `shuttleengine/fixture_test.go`

**Edit:**
- `shuttleengine/run_test.go`
- `shuttleengine/wait_test.go`
- `shuttleengine/attach_test.go`
- `shuttleengine/startup_test.go`
- `shuttleengine/session_test.go`
- `shuttleengine/run_inject_test.go`
- `shuttleengine/waiting_test.go`
- `shuttleengine/run_resume_test.go`
- `shuttleengine/gate_test.go`
- `shuttleengine/gate_list_test.go`
- `shuttleengine/gate_pending_test.go`
- `shuttleengine/attach_asking_test.go`

**Uses:**
- `shuttleengine/fakes_test.go`

**Intent:**
Give shuttleengine's tests one Runner/Run fixture (#324 §4) in place of the near-identical constructors that each make a temp worktree, create the anchor and call `NewRunner`.

`fixture_test.go` (`package shuttleengine`, untagged) defines:

- `newFixture(t, reed ReedOps, engine Engine, opts ...fixtureOpt) *fixture`, exposing `Runner`, `Anchor`, `Worktree`, `DotLyx` and `RunRoot`, with options for the variants the constructors take today (a `Config`, a `clock`, a pre-seeded strand GUID);
- `fastConfig` and `defaultConfig`, the two `Config` shapes nearly every literal uses (`{StartupTimeoutS: 30, RunTimeoutMin: 5}`, and that plus `PollIntervalMS: 1, LivenessEveryNPolls: 1`);
- `fx.newRun(spec Spec, opts ...runOpt) *Run`, which fills `Run`'s private fields (`runner`, `spec`, `runDir`, `state`, `clock`, `deadline`).

Replace the constructors with it: `newTestRunner`, `newWaitTestRunner`, `newAttachTestRunner`, `newStartupWorktree`/`newStartupTestRunner`, `newSessionTestRunner`, `newInjectTestRunner`, `newInterruptTestRun` and `newPollTickRun`; their callers use `newFixture` (and `fx.newRun` where they built a `Run`).
The `Config`/`&Run{...}` literals elsewhere are card 24's.

These are protected Completion Signal regression tests (D9): restructure only, every table row keeps its assertion.
The in-package fakes in `fakes_test.go` stay: an import cycle forbids `shuttlefake` here (D1).
Apply D10 to every touched test.

**ImpactSummary:** Untagged shuttleengine test files only; per-file Runner constructors collapse onto one fixture, no assertion changes.
