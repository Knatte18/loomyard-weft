```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked the sentences the fix added or rewrote in cards 7, 8, 23 and 25 against the code at HEAD.

no findings

- **Card 7, import cycle:** `go list -deps ./internal/reportengine` includes `internal/configreg`, so `internal/reportengine` depends on it transitively. `internal/reportconfig/doc.go:9` gives the same reason.
- **Card 7, `testmain_test.go`:** it has no build tag and declares `TestMain`.
- **Card 7, other kept claims:** `template.yaml` holds the stated line. `LoadConfig` refuses a value under 1 (`config.go:58-59`), naming the key and `lym config report`. `configStrictnessDegradingSet` holds `internal/reportconfig`. `wantHubWide` names `report`.
- **Card 8:** `reportconfig.LoadConfig` exists with the stated signature.
- **Card 23, doc lines:** the quoted lines match the files. `shedrun/doc.go:5`, `loomrecipe/doc.go:34/48/63`, `shedcli/doc.go:12` and `runprofile/doc.go:3/12` each list the recipes. `loomcli/doc.go:25` holds the quoted Sketch sentence.
- **Card 25, `pawlengine/doc.go`:** line 23 lists the four `.lym/beam/` files the card names, and line 8 is the Beam-Up description.

Tool calls:
1. grep of `internal/reportengine` imports, the `testmain_test.go` header and the `LoadConfig` refusal in `internal/reportconfig`: my context did not show the reportconfig code or the reportengine imports.
2. `go list -deps` of `internal/reportengine`, plus a grep of the two `doc.go` files: the direct-import grep found no import of `configreg`, so I needed the transitive dependency set.
