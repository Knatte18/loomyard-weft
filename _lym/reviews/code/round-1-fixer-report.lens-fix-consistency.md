```yaml
lens: fix-consistency
outcome: reported
```

Task: check the two new lost-writer subtests in `internal/seatengine/resume_test.go` for agreement with their surroundings (fix-consistency lens).

no findings

- **Siblings:** The new subtests use the same fixture and helpers as the two existing `TestEngine_Resume_Writers` subtests (`engineTable(..., 1)`, `asWriters`, `chair.sent`, `os.WriteFile` on `Seats[1].Outputs[0]`). Their names follow the existing "a writer not found whose …" pattern.
- **Behavior in `resume.go`:** The assertions match `Resume`'s `case seat.Writer` else-branch. `ender.end(lost)` fires, the lost `seatRun` has a nil handle, and its result carries empty `StrandGUID` and `Outcome`. Accept is not called when `allExist` is false, and its error is recorded on `AcceptErr` when it fails. `join` then stops the chair once (`stepEnded`) and queues no line, which is what the stop and `sent` assertions check.
- **Docs and header:**
  - The `Resume` doc comment, the `doc.go` text on writers, and the `Required`-advisor test all already describe this behavior, so no text needs updating.
  - The `resume_test.go` header ("attaching to a live chair, and telling it of the advisors the probe lost") is still accurate. A lost writer that ends the step is a case of the same probe-loss theme.
- **Twins:** The mapper test the review mentions (`TestMultiLLMProducer_MapsAWriterThatEndedTheStep`) feeds a hand-built `SeatResult`. The new subtests now check that the engine leaves `StrandGUID` and `Outcome` empty for a lost writer, so the two agree.

Tool calls: none; the context already held every changed and referenced file.
