```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran each finding's search to check the fix covers every member.

Findings left unchecked: none. I checked F1, F2 and F3.

- **F1 (open maps held in two places):** I searched the tests and production code for every reader of a module's `OpenMaps`. The registry members are `TestModules_SeedOnlyAndHubWideFlags`, which compares the Spec with the `Modules` entry, and `TestLookup`'s `wantOpenMaps`. Card 2 now covers both, through the `Modules` reed row and the new `TestLookup` row. Production code that branches on `OpenMaps` is `fabriccli/fabric.go:794`, which reads hub-wide modules only, and `configsync.go:914`, where the `!m.HubWide` branch already applies to reed. Neither needs a card change. The `configreg/fill_test.go` reed rows name explicit key paths and do not enumerate the template's leaves, so the open maps leave them unaffected.
- **F2 (Extends on lines a card rewrites):** I listed every card's remaining Extends entries: card 2 `reedengine/doc.go`, card 3 `reedengine/doc.go`, card 4 `hubgeom/doc.go` and `reedengine/doc.go`, and card 8 `reedcli/doc.go`. Each Intent says the file "gains" a new sentence or paragraph and changes no existing line. Only the two entries the review named were wrong, and both now sit under Edit.
- **F3 (stale "two-line" text):** I searched for every mention of a two-line bar or `status 2` in the code. These are `bar.go:1`, `windowsize.go:1,45`, `doc.go:683`, the `windowsize_test.go:46-51` doc comment and testtiming reason, the `bindings_integration_test.go:338,344` comment and Fatalf, `smoke_selvage_keepalive_test.go:273`, and `bar_integration_test.go:3`. Card 5 now names all of them, the last through the sentence the fix added. The other hits are unrelated (exit status 2, a two-line draft or entry).

no findings

- Call 1: searched the tests for `OpenMaps` readers. My context showed only the production grep, which excluded test files.
- Call 2: listed every card's Extends entries and its doc wording. My context held only cards 2, 5 and 9.
- Call 3: searched the repo for two-line bar and `status 2` mentions. My context held only the `bar_integration_test.go` header.
- Call 4: read `configreg/fill_test.go` to see whether the reed fill rows enumerate template keys. My context held only its `fillModules` line.
