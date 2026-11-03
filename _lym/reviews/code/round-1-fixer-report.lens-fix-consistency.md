```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: no BLOCKING findings, and two MEDIUM and two LOW where the F6 change left a twin unchanged.

**[MEDIUM] `walk.go:218` Warn text and way forward**
- The Warn reads "lacks units its template holds, and reconcile never adds them", with the way forward "add each with `lym config <module>` …".
- `LackingTemplateBlocks` now also returns `light.loom: differs from light.tapestry` (`migrate.go`, the new loop before `slices.Sort(lacking)`), and `result.Lacking` reaches this Warn.
- That entry is a conflicting pair the operator must settle, not a template unit to add, so the way forward misdirects the operator.
- `ensure_integration_test.go:1311` pins the old wording, so it must move with the message.

**[MEDIUM] `configsync.go:203` `Result.Lacking` doc and `configcli.go:630` comment**
- `configsync.go:203` says Lacking "lists the template units a seed-only module's file lacks".
- `configcli.go:630` says `lacking` is set by "a seed-only module whose file lacks a template unit".
- Both now under-describe the hook output, which also carries a differing former/current pair.
- The F2 test pins the `lacking` envelope key, so a consumer of that key meets an undocumented entry shape.

**[LOW] `migrate_test.go:17` `TestMigrateConfig` doc comment**
- It still says the test pins "the drop of a former block beside a current one".
- The rows now pin the drop of an equal block and the keeping of a differing one.
- `TestLackingTemplateBlocks`'s doc at line 103 likewise omits the differing-pair listing its new row pins.

**[LOW] `migrate.go:1` file header**
- It says the file lists "the template's recipe blocks a starting profile lacks", which no longer covers the differing-pair report.

I found no other twin: `yamlengine/rename.go` has its own "removed, … already set" line format for generic renames, and this change did not touch it. `doc.go` and the `MigrateConfig` and `LackingTemplateBlocks` docs agree with the code.

Tool calls (1 of 6):
1. `grep` for "already set", "Lacking hook", "lacks units" and related phrases across `*.go` and `*.md`: to find twin docs and messages that describe `Lacking` or the former-block drop, which my context lacked.
