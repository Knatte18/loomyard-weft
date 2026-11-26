```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F8 fix (fix-code-correctness lens).

no findings

Traced:
- **`rangeRulingKey` and `listFiles`:** a failing `RangeDiff` returns the wrapped error before any ruling is reused or stored. A range that only adds or edits a new file now changes `rangeKey`. It re-judges only the rulings `dependsOnRange` selects.
- **Callers:** the `RangeDiff` seam is set in the loom wiring, `envkit`, the `shedrecipe` fixture and the `verify_test` fixture. It is required by both seam lists and listed in the `battencli` seam list. The gate passed over those packages.
- **New uncaused-ruling test:** with the defect in place (key built from `ChangedFiles`), neither the files nor the trailers change on the second pass. The ruling is reused, the pass is Stuck and `requireDone` fails.
- **Added-carrier test:** with the defect in place no second judge runs, so `f.seats.Calls != 2` fails. The short circuit keeps `f.requests[1]` from being indexed.

Further tool calls: none.
