# Fixer report — plan, round 1

## Changes

- **F1** (fixed as proposed): card 6's `ArbiterSeam` gains `AttemptEnvelope`, which `resumeOpenAttempt` calls for an open arbiter directory when the tackler attempt yields no envelope path; with `Arbiter` nil an open arbiter directory is ignored and the driver steps.
  The loomcli wiring paragraph fills it from `loomengine.ReadArbiterAttempt`'s `EnvelopePath`, and `TestGoDriver_Settle` gains the arbiter-only restart row.
- **F2** (fixed as proposed): card 4's recipe seams make `TacklerDeps.Snapshot` optional, with `settleFresh` recording an empty `WorktreeSnapshot` when it is nil, and `TestTacklerCall_SpawnPath` gains the nil-`Snapshot` row.
  Card 6's tamper bullet now points at card 4's behaviour instead of prescribing it.
- **F3** (fixed as proposed): card 13 lists `contracts/specs/refusal-spec.md` under Edit, and its run-verb bullet adds the shed-section row "batten run: a batten driver already drives the run" (transient, way forward `lyx batten status <slug>`), the input that reaches it and the driver-lock row of `TestRunCmd_HeldRunLockRefusesNamingTheLockPath` that asserts `battenDriveRefusal`.
- **F4** (fixed differently, in part): card 8's Validation bullets have `battenshed.LoadConfig` refuse a resolved document holding `driver_kind`, naming `batten config key "driver_kind"`, the reason and the way forward, and battenshed's `TestLoadConfig` gains the base-layer row.
  I did not add the proposed claim that "the refusal-spec row card 8 already edits covers it": no row in `contracts/specs/refusal-spec.md` covers a batten `driver_kind`, since the only `driver_kind` row is loom's "config: driver_kind not llm or go" (line 258), whose trigger is a value that is not `llm` or `go`, not a key batten refuses.
  Instead, card 8, which already edits `refusal-spec.md`, adds a shed-section row "config: driver_kind in batten.yaml", class correctness halt, as `PATTERN-refusal-way-forward` requires for a new refusal.
- **F5** (fixed as proposed): card 4's Edit bullet re-signs `internal/loomcli#arbiterFake.Start` to return `sheddriver.AgentRun`.
- **F6** (fixed as proposed): card 7 drops the `TestResolveDriverKind` Create bullet and moves its three rows into `TestResolveTackler`'s table.
  The sheddriver budget figures fall by one: card 7 now goes 15 to 18, card 10 goes 18 to 19, and the overview's test-budget-rows decision reads 3, 9, 15, 18, 19.
- **F7** (fixed, wording refined): card 11's `IdleTimeout` bullet states the bound on a wedged batten step: the loop kills none, as today's generic run body kills none, and `lyx batten pause <slug>` or ending the detached loop job bounds it.
  The proposed text applied "Run-Shed's reinvoke policy" to every killed step, but card 11's own `battenInterruptPolicy` hands back at every row except Run-Shed, so the bullet says the killed step follows its row's policy: a re-step under the repair cap at Run-Shed and a hand-back at any other row.
- **F8** (fixed as proposed): card 13's Tests list adds the paused `TestRunCmd_ResumeDispositions` row, and the claim now names the done and paused envelopes only, saying why no run test drives a blocked stop.

`lyx loom validate-plan` passes (`ok: true`).
Its only new informational finding, `caller-uncovered` on `arbiterFake.Start`, matches `.Start` calls in `internal/proc` and `internal/vscode`, which are not callers of the test fake.

## Disputed

None.

## Deferred

None.
