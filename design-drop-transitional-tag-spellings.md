# Drop the transitional tag spellings after the deploy

`drop-transitional-tag-spellings` · Task · enhancement · done

## Drop the transitional tag spellings (D13 follow-up of `gate-runs-once`)

`gate-runs-once` retags every test file to its one `tierN` tag.
Until the deployed `lyx` replaces the installed guard scan, a few spellings stay beside the new tags.
After the deploy, remove each of these:

- the guard files' `|| integration` term, and the one-tag scan's allowance for it in `cmd/lyx/tiertags_test.go`;
- the old tag spellings in the hub's tier commands;
- the old tag spellings in the package loads of `internal/impactset` and `internal/planglyph`;
- the old tag spellings in `cmd/testtiming`'s spawn scan and in its `defaultRedundancyTags`;
- the old tag spellings in the two guards that refuse `tier2` and `tier3` as `tmux` and `llm`;
- the old tags in the tier purity guard and in the tmux isolation guard's `isolationTags` and `isolationTagSets`;
- the `tmux` and `llm` terms of `internal/testkit/tmuxkit/allow_tmux.go`'s constraint;
- the `integration` spelling in `internal/planglyph/testdata/glyphchain/shapes/tagged.go`, which becomes `tier1`.
