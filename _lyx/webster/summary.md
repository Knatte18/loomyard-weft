# Engine registry: shuttle picks each run's engine from a registry

All four batches (14 cards) reached `done`, and no recovery was needed.
The scope guard listed files outside each batch's own targets as deviations; they are informational, and none made a batch stuck.

## Batch 01-engine-set (cards 1-6)

Head `865f259d9`.
Added `EngineSet`, `shuttle.yaml`'s `engine:` key (an empty value is refused), the `Requirement` type and the optional capabilities as implemented by Claude, `ModelChoice`, and the `internal/shuttleengines` registry.
`SessionNamer` moved into the registry, and the three cross-run readers now dispatch per run record through the set.
Deviations the digest listed: `internal/cliwire/sessionnamer.go`, `internal/loomcli/reviewstatus_test.go`, `internal/tendercli/deps_test.go`, the `internal/shuttleengine` and `internal/shuttleengines` test files, and `engine.go` and `engineset.go`.
The fork reported editing three existing tests that no card listed, and adding `internal/shuttleengines/testmain_test.go`.

## Batch 07-runner-engine-set (cards 7-8)

Head `d10eff045`.
The `Runner` constructors take an `EngineSet` at every call site.
`Start` and `Open` resolve the engine by spec and record its name; every other operation resolves it by the run's record.
Deviations: `internal/shuttleengine/enginedispatch_test.go`, `internal/shuttleengine/open_test.go`, `internal/webstercli/wiring.go`.

## Batch 09-guid-engine-reads (cards 9-11)

Head `0b06238a0`.
`InterruptSettle` and the guid-less session reads take a guid, through orch and tender.
`Spec.Requires` is checked at Start and attach.
Fork audit moved off `Engine` and is asserted in Webster's wiring.
Deviations: `internal/orchengine/watcher.go`, `internal/tenderengine/report.go`, `internal/shuttleengine/engine.go`, `requires.go`, `requires_test.go`, `run_test.go`, `completionsignal_enforcement_test.go`, `enginedispatch_test.go`, `internal/shuttleengines/shuttleengines_test.go`, `internal/websterengine/auditengine_test.go`.
The fork reported fixing a flaky test it had introduced in `enginedispatch_test.go`.

## Batch 12-orch-resume-context (cards 12-14)

Head `10a31f018`.
The orch resume context renders through the engine capability, `ModelSwitchSequence` is deleted everywhere, and `PATTERN-shuttle-provider-seam` is amended with a widened scan.
Deviations: `internal/shuttleengine/claudeengine/modelswitch_test.go`, `startup.go`, `internal/shuttleengine/engine.go`, `seam_enforcement_test.go`, `pattern/PATTERN-pane-text-injection.md`.
The fork reported that card 12's `resumeContextCmd` resolves the capability before calling `ResumeContext`, and that the `PATTERN.md` entry for the seam is shorter than the card's wording to fit `lint-pattern`'s headroom.

## Not run

The forks compile-checked the `tmux` and `llm` tiers only and did not run them.
The plan-level verify gate runs after this summary.
