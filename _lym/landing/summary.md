# testcover: shared coverage engine, content block keys, map record

`internal/testcover` is new.
It holds the per-test coverage machinery that `cmd/testtiming` owned, so the tool and later callers share one runner and one verdict.
A `main` package cannot be imported, which is why the code moves rather than being called in place.

## What changes

- `internal/testcover` owns:
  - the coverage-binary build and the run-alone runner (`Toolchain.BuildBinary`, `ListTests`, `RunAlone`);
  - profile parsing and the package-block filter;
  - `Judge`, the contained and greedy removable verdict;
  - the static spawn scan and the `//testtiming:keep` scan, moved with `git mv` from `cmd/testtiming`;
  - content block keys;
  - the per-package coverage map record, `Map`, with `ReadMap` and `WriteMap`.
- `cmd/testtiming -redundancy` keeps one coverage binary per package, instrumented over the whole module, but judges each test by the blocks of the package's own files.
  Fixture-package blocks no longer make nearly every test unique, so packages such as `internal/fabricengine` can now list candidates.
  Each candidate row gains a "Contained module-wide" column; a cut contained under both views is the safer one.
- `-pkg` and the other flags are unchanged.
  A `-pkg` splice now also rewrites the report header to the current one; unmeasured sections stay byte-identical.
- Eligibility is unchanged: a skipped, failed, coverage-free or possibly out-of-process test is not judged.
  "Covers nothing" now means nothing of its own package.
  A kept candidate is still listed under Kept.
- `internal/lymdirs` gains `TestCoverDirName`, the directory under the hub's `.lym` for the maps.
- `cmd/lym`'s tier-purity guard is re-pointed at the moved `spawnscan_test.go`.
- `docs/overview.md` and `PATTERN-test-speed.md` describe the new package and the extra column.

## Design decisions

- Block key: package-relative file, enclosing function (receiver type for a method, a fixed name outside any function), a hash of the block's source text, and an ordinal among blocks equal in those three.
  Line shifts keep a key; an edit gives a new key that no map test covers, so an edit errs towards no finding.
  Bound: inserting a block that repeats the text of another in the same function shifts the ordinals of later identical blocks, which can give a wrong cover and a false contained verdict.
  A reviewer reading each finding against its covering tests bounds that.
- Map record: one JSON file per package under `<hub>/.lym/testcover/<package>/map.json`, shared by every worktree of the hub on the machine.
  It holds the commit, build time, per-file content hashes, the block key table, and per judged test its tier, function hash, duration and covered block indexes, plus any build failure.
  It is read and written through `internal/state`, keeps unknown keys, and replaces the old record whole.
- The runner takes no gate slot; its caller holds one.
- Nothing in this change writes or reads a map yet: the record type and its I/O are exported for later callers.

## Tests

No new test launches a test binary or `go test`.
The runner's tests use a stand-in `go` and fake profiles.

## To check by hand

Nothing was left unrun.
A real `go run ./cmd/testtiming -redundancy -pkg ./internal/fabricengine` run would show the new module-wide column on live data.
