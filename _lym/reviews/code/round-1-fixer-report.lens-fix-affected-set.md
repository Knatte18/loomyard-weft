```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set, over the F1, F3 and F5 premises (the findings whose premise is a set of sites). Findings left unchecked: F2, F4, F6, F7, F8, F9, F10, which have no set premise.

No findings.

- **F1:** `replayFields` now carries a `record_body_sha256` row. I looked for other field tables or per-field comparisons a new `Task` field must join. `runlock.go:211` already lists `RecordBodySHA256` as never compared. `endsAsFamilyProject` (`raddlefile.go:261`) compares only `Discussed`, `Status`, `Profile`, `RunSettings` and `NotifyOnLand`. The stamp is stored and carried but never compared there, so no site was missed.
- **F3:** The only site that carries the discussed flag across a remove-and-re-upsert is `store.go` (the `keepDiscussed` branch). It now carries the stamp as well. The other `.Discussed` writers each set or clear the flag as part of storing a record: `raddlefile.go:197` and `decision.go` set the stamp beside it, and `render.go:72` is a render-only copy. No other path drops the stamp.
- **F5:** The only remaining "once origin answers" strings are in other modules (`battenshed`, `landingshed`, `adoptcli`, `battencli`). Their origin-unreachable premise is true there, so they are out of scope.

Tool calls beyond context:
1. Grepped `.Discussed` uses in non-test code, to find every site that carries the flag and might need the stamp too.
2. Grepped `"discussed"` and `beam_color` string literals, to find other field tables like `replayFields`.
3. Read `raddlefile.go:252-270` and `runlock.go:205-225`, to see whether any field comparison needed the stamp; context had not shown them.
4. Grepped `record_body_sha256` in `doc.go` and the rename files, to check the documentation and `rename` for stamp handling; `rename` has none.
