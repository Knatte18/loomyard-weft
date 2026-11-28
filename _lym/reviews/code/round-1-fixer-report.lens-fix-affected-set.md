```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix for each set-premised finding (F1–F5) covers every member of its set.

**Findings left unchecked:** none.

**no findings**

- **F1:** every key in the `doc.go` key paragraph is now written in full. Bare names remain at `config.go:46` (`verify_gate`) and `verify.go:297` (`fixer_timeout_min`); the fix did not touch or reach either, so I don't raise them.
- **F2:** both tests the review named now assert `Kind`, `Start` and `End`. The other `spanKinds` users compare kinds on purpose.
- **F3:** the only place that records a rework span is `runFixer`. A `SpawnOrAttach` error returns before the span, and `gatefixer` abandons the strand on its post-start failures.
- **F4:** the attempt-timeout comments in `config.go`, `template.yaml`, `strand.go` and `doc.go` agree. The fixer-timeout "after spawn" comments are correct to leave, because `gatefixer.Await` measures from `rec.SpawnedAt`.
- **F5:** `LiveFixerRecordIsAttachedDeliveredToAndAwaited` is the only resume row that attaches to a fixer, and it now asserts the spans.

Tool calls:
1. Grep of config key names in `weaverengine` comments (F1): my context had no full set of comment mentions.
2. Grep for attempt-timeout and "after spawn" wording (F4): needed for sites outside the files already read.
3. Grep for span and attach test sites (F2, F5): needed the post-fix list, including `shedadapters`.
4. Grep for `SpawnOrAttach` and `Await` in `gatefixer` (F3, F4): the implementation wasn't in context, and this call returned only the wrappers and fakes.
5. Grep for the package functions `SpawnOrAttach` and `Await` (F3, F4): needed their bodies.
