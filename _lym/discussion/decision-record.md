## Goal

```markdown
## Seen (remove-launchers-dir, 2026-11-11)

The verify gate fails on `internal/loomcli` `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why` (`smoke_test.go:738`), and it reproduces on a rerun.
The driver log now carries `shedverbs: step interrupted (loop-exited)`, where the test waits for the poisoned status file's `invalid state`.
The wording comes from main's `d1535412` (shedverbs: record how an `--until-stop` loop's process ended); the run changed no shed, loom or driver code.

## Idea

Decide which message the died-driver case should surface: the loop's recorded exit reason, or the invalid state of the poisoned status file.
Then change the code or the test to match, and keep the reason the run halted in the driver log.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
