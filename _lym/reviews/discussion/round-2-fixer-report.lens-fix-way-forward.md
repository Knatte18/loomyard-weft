```yaml
lens: fix-way-forward
outcome: reported
```

I checked the ways forward this round's fix added to `_lym/discussion/decision-record.md`.

**Ways forward the fix added:**
- **D5, retired-key sentence (from F4):** a hand-edited local file holding `webster_reviewer_fan` or `webster_fan` is refused at load until a reconcile migrates it.
  This can be reached and makes progress.
  `reconcileSparseFile` (`internal/configsync/configsync.go:547-575`) reads the raw file with `os.ReadFile` and runs `m.Migrate` with no `Load`/`ValidateLocal` first.
  So a file whose load refuses is still migrated onto `code_reviewer_fan` by a reconcile.
  The rename also has to be written to disk when apply is set; it is, because `changed` counts `Migrated`.
- **D2 (from F1):** `way-forward`'s prose gains the emphasis line.
  The edit can be carried out: the line is appended to the lens text, and `TestConfigTemplate` then passes it as a non-procedure lens.
  Nothing blocks it.
- **D5, third Bound bullet (from F2):** an agent can point the code review at another fan.
  A wrong name is refused by `ValidateFanKeys`, which names the fix: define the fan in `lenses.yaml` or set the entry to `""`.
  This leads somewhere that makes progress.

No findings.

One thing outside my lens, for fix-claims: "as for every retired key" in D5 is broader than the code.
Keys the spec itself lists as retired with category none are admitted at load by `ValidateLocal`'s `retiredCovering` `continue`, not refused.

Tool calls:
1. I grepped `configsync` for `Migrate` and the reconcile functions, because my context did not show whether a reconcile runs `Migrate` without a validating load first.
2. I read `reconcileSparseFile` (`configsync.go:547-600`) to confirm it migrates the raw file before any validation and writes the result.
