---
round: 1
focus:
  - "internal/testcover/blockkey.go and blockkey_test.go: settle whether the key holds the four D3 parts (package-relative file, enclosing function with receiver type for methods or a fixed name outside functions, source-text hash, ordinal among blocks sharing the first three) and whether line shifts keep a key, edits change it, and same-named methods on two receivers differ."
  - "internal/testcover/verdict.go against cmd/testtiming/redundancy.go: settle whether the moved containment and greedy pass behave as before, and whether the package view keeps only blocks in the package's own files while the module-wide verdict is still computed for each candidate (D2)."
  - "cmd/testtiming/redundancy.go and main.go: settle whether the flags and the -pkg splice of only the named packages' sections are unchanged, whether the report shows the module-wide column, and whether a kept candidate is still listed under Kept with its classification unchanged while being dropped from judging."
  - "internal/testcover/runner.go and profile.go: settle whether each top-level test runs alone and serially with its own profile, whether the build instruments the whole module for the tool but only the package for the map (-coverpkg), and whether skipped, failed, coverage-free and spawn-scan-flagged tests are never judged."
  - "internal/testcover/coveragemap.go with internal/lymdirs/dirs.go: settle whether the record carries every D4 field (commit, build time, per-file hashes, block key table, per-test tier, function hash, duration, block indexes, build failure), is written and read through internal/state, keeps unknown keys, replaces the old record whole, and sits at a path lymdirs names under the hub's .lym tree."
  - "internal/testcover/spawnscan.go and keep.go moves: settle whether the spawn scan and keep scan moved with unchanged behavior, whether exports match the Exports line, and whether cmd/lym/tierpurity_test.go is re-pointed at the moved files with no stale reference to cmd/testtiming's copies."
  - "Test files in internal/testcover and cmd/testtiming: settle whether any test launches a real test binary or go test (the constraint requires a stand-in go and fake profiles), whether each acceptance criterion's behavior is covered, and whether any new test duplicates one the move kept."
  - "docs/overview.md and the repository's constraints and PATTERN entries: settle whether the docs the change requires landed and whether the diff honors the listed patterns, including comment conventions on new doc comments."
---

Seed focus for round 1; the diff is the single commit pair that introduces `internal/testcover` and re-points `cmd/testtiming`.
