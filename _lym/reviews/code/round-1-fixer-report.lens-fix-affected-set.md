```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the sets behind F1/F2, F5, F6/F7 (every site that states the reconcile-only rule, the old wording, or the callers) after the fix.

No findings.

- **F1/F2 (reconcile as the only removal):** a search of code and docs, outside `_lym`, `.lym` and `docs/research`, for "only/one way … reconcile … remove/prune" found no remaining sentence. `PATTERN.md:16` does not mention reconcile, so it still matches.
- **F6 (old load-failure wording):** no remaining occurrence of "would not load with this value" or "--set the key to a value" in code or docs. The two test strings that held it were updated.
- **F7 (`decodeGateKey` callers):** the only callers are the two in `reinterpret.go`, and both use the new two-value signature.
- **F5 (`LoadRefuseRaw` callers):** `claudeengine.go:163` and `codexengine.go:171` handle the returned error the same way as before. `internal/gateslot/doc.go:70` describes the fallback without saying which errors it refuses, so the changed doc comment does not contradict it.
- **Reinterpretation wording elsewhere:** `docs/overview.md:515` and `configcli.go:513` name the `--unset` route and agree with the flagged way forward.

Calls made:
1. Searched the repo for sentences naming reconcile as the sole removal route, to cover F1/F2.
2. Searched for the old load-failure wording, the old "names no scope" text, and the `LoadRefuseRaw` and `RetireHardenLiveCopies` callers, to cover F5/F6/F7.
3. Read `internal/gateslot/doc.go:66-80` and grepped the docs for `harden-live-copies` statements, because my context did not hold the package doc or the docs that describe the reinterpretation.
