---
round: 2
focus:
  - "Production Go comments under cmd/ and internal/: does any still name a config key short of `<module>.<dotted path>`? Search by phrasing as well as by key name: `<file>.yaml's <key>`, ``<file>.yaml's `<key>` ``, `<module>'s <key>`, `its dobby's <key>`, `the <module> <key>`, and one-word template keys such as autonomy, squash, engine, scoper, ladder and watchdog in backticks or after a possessive. Sites to start from: internal/landingshed/finalize.go, internal/landingshed/finalize_pr.go, internal/battencli/dobbyreads.go, internal/hubgeom/autonomy.go, internal/hubgeom/doc.go, internal/shuttleengine/codexengine/codexengine.go."
  - "`_test.go` comments under cmd/ and internal/: the task asks for every Go comment. Does any test comment still name a config key short of its full name, for example `shed.yaml's driver_kind` in internal/battencli/childdriverkind_test.go or `beam.yaml's pull_request` in internal/landingshed/prreview_test.go? Did any test-data literal change?"
  - "Comments the range rewrote: does each still say what the code does, with no word lost in a line split? Check internal/dobbyengine/reinterpret.go, where the retired board.json entry keys pull_request and notify_on_land (see internal/boardengine/store.go) must not read as the live beam config keys, and where the comments on PerTaskSettingsReinterpretationID and ReinterpretPerTaskSettings must be whole sentences."
  - "internal/configreg/keynames_enforcement_test.go, bareKeyNames and stringLiterals: is each skip rule (colon, `.`/`%`, `--set`/`--unset` prefixes in both quote forms, one-word, whole-literal, KeyName call, struct tag, import path) held by a test with literal expected values, and does any skip let a message such as `<key>: <problem>` name a key bare and pass?"
  - "internal/configreg/keynames_enforcement_test.go, bareKeyNameAllowlist: can a new literal in an allowlisted file name the same key bare and pass?"
  - "_lym/landing/summary.md: does the diff bear out each claim on which comments were swept and whether any changed meaning, and does the Merge danger blast radius name everything the diff can break, including the `lym config` help text and the new tier0 guard on production string literals?"
---

Each entry names an area that the next round's changes are expected to touch or that needs a fresh look after them.
