```yaml
lens: code-pattern-fit
outcome: reported
```

Task: lens code-pattern-fit. I checked the diff against the PATTERN entries it touches and the CLAUDE.md rules.

**Rules checked**
- `PATTERN-cwd-resolution`: held. `RepositoryRootOf` adds only an `os.Getenv` read and path-list parsing. It adds no cwd query (`internal/cliwire/paths.go:127-158`).
- `PATTERN-test-speed` and `PATTERN-tier-tags`: held. The new row has `t.Parallel()`. It drives the unexported walk, so it needs no `t.Setenv`. `cliwire_test.go` keeps its `tier0` tag. `TestSetEnv` keeps its existing `t.Setenv` use and its tag.
- `PATTERN-testkit`: held. The `tmuxkit` change adds one `os.Setenv` inside the existing `setTempEnv`. It adds no import and no spawn.
- `PATTERN-comment-line-breaks`: held on my read. Each new sentence sits on its own line. The two-line `RepositoryRootOf` paragraph breaks at a colon, not mid-phrase. The gate's `lyx loom lint-comments` is the actual check.
- Comment conventions: held. The new doc comments are self-contained, and neither new helper has an end-of-line comment.
- `PATTERN-lyx-temp-root` and `PATTERN-lyxdirs-single-declarer`: not touched (no temp creation, no `_lyx` or `.lyx` path).
- `PATTERN-test-isolation`: touched, and see finding 1.
- Declared PATTERN changes landed: the directive renders no declared changes list, so there is nothing to check.
- Commit messages and departures: the three commits carry no `Pattern-Departure:` trailer, and none is needed because no pattern entry forbids the answer key's way.
- CLAUDE.md "Docs land in the same commit": the module doc for `RepositoryRootOf` (the extended comment) and the `tmuxkit` package doc both landed. A grep of `docs/*.md` and `internal/cliwire/doc.go` found no `RepositoryRootOf` statement that needs updating.
- Scope against the board entry and decision record: the diff makes only the changes the record lists, and I found no extra change.

**Findings**

1. LOW, class `consistency`: `pattern/PATTERN-test-isolation.md` was not updated for the new `GIT_CEILING_DIRECTORIES` environment setup.
   - Its "Private temp directory and leftovers" section lists what `tmuxkit.Main` sets: `TMPDIR`, `TMP` and `TEMP`, and `TMUX_TMPDIR`. The diff adds a fourth variable, set on every platform, that changes how git and `cliwire`'s walk behave in every test package.
   - CLAUDE.md says a new cross-cutting invariant goes into PATTERN in the same commit.
   - The record's constraints and notes name no PATTERN change, so this may be judged as no new invariant. The background file otherwise describes `Main`'s environment in detail, so the omission leaves it stale.
   - Evidence: `pattern/PATTERN-test-isolation.md`, "Private temp directory and leftovers", first bullet. `internal/testkit/tmuxkit/tmuxkit.go:461`.

**Tool calls**
1. One combined `Bash` call. It read `pattern/PATTERN-testkit.md` and `pattern/PATTERN-test-isolation.md`, which the manifest lacked. It also grepped `internal/cliwire/doc.go` and `docs/*.md` for a `RepositoryRootOf` or `.git` statement that needed updating. It also read the first lines of the two test files to confirm their tier tags.
