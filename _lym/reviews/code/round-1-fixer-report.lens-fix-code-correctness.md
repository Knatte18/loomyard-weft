```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed.

No findings.

I traced these by reading, without running mutations:

- `refuseBeamCancel` with and without remove, held and unheld, including a failing `LandBlockers`.
- `workingBlockers`.
- Both wordings of `heldBeamCancelRefusal`.
- The moved `EndBeam` block in `clearRunPhase`, with a failed push, an owed push and a failed flag clear.

The new table rows and integration subtests would each fail with their defect in place. The `StopBeam` guard case rests on `BeamDown` refusing an unmarked pair, as the change description states.

Outside my lens: the order sentence in `internal/battencli/cli.go`'s package doc ("remove the records; leave the claim draft; lift …; push and notify") does not mention that the beam flag is now cleared between the push and the notify.

No further tool calls.
