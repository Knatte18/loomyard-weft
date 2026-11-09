```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: fix-consistency lens over the F1–F4 change set (commits fbc13466, 7f83eaa1, 88ec7cd2, b2613f4f).

- **F3 field comment:** the reworded `Spec.JSONEntries` comment at `layers.go:78` agrees with its twins, which already name the undotted/dotted split. Those are `configengine/doc.go:27`, `configengine/set.go:48`, `configengine/set.go:110` and `yamlengine/doc.go:13`.
- **F4 doc sentence:** the new sentences in `configcli/doc.go:138` and `yamlengine/set.go:846` say the same thing and match the dotted-key paragraph they follow. No other doc describes dotted-key handling without the new rule.
- **F1 and F2 rows:** the new test rows follow the existing table conventions.
- **No stale sibling:** the grep for the old "each pair as one whole top-level entry" wording found no remaining instance.

Tool calls made:
1. `grep -rn "whole top-level|dot in a key|JSONEntries"` over non-test `.go` and `.md` files, to find twin comments the context didn't show (`configengine/doc.go`, `configengine/set.go`, `yamlengine/doc.go`).
