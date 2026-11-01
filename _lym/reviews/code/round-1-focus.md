---
round: 1
focus:
  - "internal/lymcwd/anchor.go and internal/fabricengine/junctionnames.go: the record says the value lives in `fabricengine.BoltDirName`; the diff changes `lymcwd.BoltDirName`, which `fabricengine.BoltDirName` aliases. Settle whether `_bolt` is the one value every consumer reads and no second spelling of the directory name remains in lym code."
  - "The scripted swap (a `_board` not preceded by a letter or digit becomes `_bolt`): check the non-Go changes (docs, SVG, calls.tsv and other research records, corpus transcript.jsonl, mill-config.yaml, sandbox suites) for a replacement that alters a recorded measurement, a quoted historical output, or an unrelated identifier, and for a `_board` spelling the swap left behind outside tools/boltrename."
  - "tools/boltrename/main.go: does it rename the checkout, the weft's worktree admin directory, both gitdir links and the hub's process records, and re-point absolute paths in config, reed.json and the VS Code files, as the Design asks? Look for a path left pointing at `_board` and for a rewrite that would also touch a name like `custom_board`."
  - "tools/boltrename/main.go: the refusal to run while any lym process or run is alive. Check how liveness is decided, that a failed or ambiguous check refuses rather than proceeds, and that the program stops with what it found when the hub is not as expected."
  - "tools/boltrename/main.go: the Design says the program stays short with no what-if code. Check for fallbacks, retries, partial-failure recovery or handling of hub layouts beyond the Neve and OSL-1046 hubs, and for a half-renamed hub state when a step fails midway."
  - "Scope: lym code must carry no fallback to, detection of, or migration for `_board`. Check that nothing outside tools/boltrename reads the old name, and that no refactor rides along with the rename."
  - "tools/boltrename/rename_test.go and testmain_test.go: test economy. Does a test duplicate another's coverage, and is any refusal or stop path of main.go left without a test?"
  - "Commit message claims (the two commits in the range) against the diff, and any `_lym/landing/summary.md` claim."
---

The Design in the record is the whole answer key: the other record sections are empty.
