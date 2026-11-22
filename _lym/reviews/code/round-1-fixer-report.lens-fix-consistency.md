```yaml
lens: fix-consistency
outcome: reported
```

No findings. This lens made 2 further tool calls.

I looked for twins of the changed text and found none left stale:
- **Reconcile and `--unset` wording:** `internal/configcli/doc.go:60,70`, `internal/configcli/configcli.go:714`, `internal/configengine/doc.go:72`, `internal/configengine/set.go:111`, `docs/shared-libs/configengine.md:255` and `internal/batcher/config.go:61,63` already name both routes or still say something true.
- **Old strings:** the old "names no scope" and "would not load with this value" strings appear nowhere outside the updated tests.
- **Launch guard docs:** `internal/shuttleengine/claudeengine/doc.go:259,265` and `internal/shuttleengine/codexengine/codexengine.go:128` match the `LoadRefuseRaw` fallback and its refusal of an invalid `refuse_raw`.

Calls made:
1. A grep across `.go`, `.md` and `.yaml` files for sibling phrasings of the reconcile-only rule and the old reinterpretation and load-failure strings. My context held only the files the findings named.
2. A grep of the engine files and docs for `refuse_raw` and `harden-live-copies` mentions outside the edited files, for the same reason.
