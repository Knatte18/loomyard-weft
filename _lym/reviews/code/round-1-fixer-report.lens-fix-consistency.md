```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Searched for twins of every changed text:
- **F1:** no leftover reference to the card fixer's hold notifier, `NotifyAsker` or `<shortname>:GATE` remains in `internal/gatecli`, `internal/gatefixer`, `contracts/orchrole` or `docs/overview.md`.
- **F3:** every statement that the notice goes out once per reach now carries the re-call rule: `orch-role.md`, `loomcli/doc.go`, the `rowNoticeProducer` and `verdictCount` comments, `shedrun.RowNotice`, `config.go`, `template.yaml` and `docs/overview.md`. The remaining "once per visit" and "within the same visit" phrases, including the `rownotice_test.go:59` comment, still read correctly because the visit is defined as the verdict count.
- **Unchanged text:** the `shedrun/doc.go` paragraph on `RowNoticeRecord` makes no once-per-reach claim, so it needs no edit.

Not checked: whether a test or golden file pins the `template.yaml` comment text or its stencil hash.

Tool calls (2):
- `grep` for the once-per-reach phrasings and the gate-notice terms across `.go`, `.md` and `.yaml`: my context held only the files I read, not the rest of the repository.
- `grep` for notifier terms plus `sed` of `shedrun/doc.go:76-82`: to confirm no other twin of the F1 or F3 text remains, and to see the `RowNoticeRecord` paragraph.
