# Card 2 — toolfloor-package

**Create:**
- `internal/toolfloor/doc.go`
- `plan:internal/toolfloor#GitFloor` -> `const GitFloor = "2.43.0"`
- `plan:internal/toolfloor#TmuxFloor` -> `const TmuxFloor = "3.4"`
- `plan:internal/toolfloor#FloorError` -> `type FloorError struct`
- `plan:internal/toolfloor#FloorError.Error` -> `func (e *FloorError) Error() string`
- `plan:internal/toolfloor#CheckGit` -> `func CheckGit(versionLine string) (parsed bool, err error)`
- `plan:internal/toolfloor#CheckTmux` -> `func CheckTmux(versionLine string) (parsed bool, err error)`
- `plan:internal/toolfloor#TestCheck` -> `func TestCheck(t *testing.T)`
- `plan:internal/toolfloor#TestLeafInvariant_StandardLibraryOnly` -> `func TestLeafInvariant_StandardLibraryOnly(t *testing.T)`

**Edit:**
- `pattern/PATTERN-leaf-packages.md`
- `docs/overview.md`
- `docs/shared-libs/README.md`

**Intent:** D4 and D5: a new leaf package `internal/toolfloor`, importing only the standard library, declares the toolchain floor, git 2.43.0 and tmux 3.4, parses a tool's version line and returns the floor error.
It holds no cache, spawns nothing and logs nothing: gitexec (card 4) and reedengine (card 6) run the probes and own their process caches.

- `FloorError` carries `Tool`, `Found` and `Floor` strings and is its own type, never a `*gitexec.GitError`, so no caller reads it as git's answer.
  `Error` names all three and the way forward, in the shape `git 2.42.0 is below lyx's minimum 2.43.0: install git 2.43.0 or newer`.
- `CheckGit` reads `git --version` output: the first dotted run of digits after `git version ` is the version, so `git version 2.45.1.windows.1` reads 2.45.1 and `git version 2.50.1 (Apple Git-155)` reads 2.50.1.
  `CheckTmux` reads `tmux -V` output of the shape `tmux <major>.<minor>` with an optional trailing letter, so `tmux 3.4a` reads 3.4.
  Both compare numerically component by component, a missing patch counting as 0.
  A parsed version at or above the floor returns `(true, nil)`, a parsed version below it `(true, *FloorError)`, and a line that does not parse, such as `tmux next-3.6`, `tmux master` or an empty line, `(false, nil)`, so the caller admits it and warns.
- `TestCheck` is one table over both functions, each row naming the function, the line, and the expected `parsed` and floor refusal:
  git admits `git version 2.43.0`, `git version 2.53.0`, `git version 2.45.1.windows.1` and `git version 2.50.1 (Apple Git-155)`;
  git refuses `git version 2.42.0` and `git version 2.39.5 (Apple Git-154)`;
  git admits `git version 3.0.0` (a higher major with a lower minor) and the two-component `git version 2.43` (a missing patch counts as 0);
  tmux admits `tmux 3.4`, `tmux 3.4a`, `tmux 3.6`, `tmux 3.10` (a multi-digit minor compared as a number) and `tmux 4.0`, refuses `tmux 3.3a`;
  both admit an unparseable line with `parsed` false.
  A refusal row asserts with `errors.As` that the error is a `*FloorError` whose `Tool`, `Found` and `Floor` are exact, and that `Error()` contains the tool, the found version, the floor and `or newer`.
- `TestLeafInvariant_StandardLibraryOnly` calls `scankit.AssertImportAllowlist(t, "internal/toolfloor")` with no extra entry, as `internal/sleepjournal`'s leaf test does.
- `doc.go` documents the package, the floor and why it equals the oldest versions lyx is proven on, the unparseable-line admission, and the D5 audit result: every git subcommand, flag or config key and every tmux command, option and format feature lyx uses (production code, test kits and `tools/`) that arrived after git 2.20 or tmux 3.0, each with the version it needs, the two forms found above the floor recorded as found and to be replaced, and the rule that a new feature must exist at the floor.
  At this card's commit the audit sentence makes no none-newer-than-the-floor claim, so it is true there; card 3 records `cherry-pick --empty=drop` as replaced when it removes it, and card 8, the last card to remove such a form, rewrites the sentence to the final none-newer-than-the-floor statement.
  The audit found these git features: `branch --show-current` (2.22), the `%(worktreepath)` format atom (2.23), `cherry-pick --skip` (2.23), `init --initial-branch` (2.28), `worktree repair` (2.29), the `maintenance.auto` key (2.29), the `receive.procReceiveRefs` key a test sets (2.29), `rev-parse --path-format=absolute` (2.31), the `push.autoSetupRemote` key (2.37), `merge-tree --write-tree` and `--no-messages` (2.38), and `worktree add --orphan` (2.42); `cherry-pick --empty=drop` (2.45) is the one above the floor, recorded as found and to be replaced by card 3.
  The tmux features: pane options and pane user options through `set-option -p` and `show-options -p`, the status-format list and range markup (`list=`, `range=window|`, `range=session|`, `norange`, `nolist`), `window-size latest`, the `-f` filter of `list-sessions`, `break-pane -b`, the `#{T:}`, `#{E:}`, `#{e|…}`, `#{>:}`, `#{&&:}` and `#{||:}` format modifiers, user ranges (`range=user|`) and the `allow-set-title` option; the two-format `#{S:}` loop is the one form 3.4 does not run, recorded as found and to be replaced by card 8.
  The implementer confirms each version against git's release notes (`/usr/share/doc/git/RelNotes/` on this host) and tmux's upstream `CHANGES` file (https://raw.githubusercontent.com/tmux/tmux/master/CHANGES, since this host ships none) before writing it, writes the version each needs as found there, and halts with a note naming any feature other than those two forms whose confirmed version is above the floor instead of listing it as satisfied.
- `pattern/PATTERN-leaf-packages.md` gains an `## internal/toolfloor` section: imports only the standard library, holds the floor and the version-line parser, and spawns, caches and logs nothing.
- `docs/overview.md`'s module tree gains the line `internal/toolfloor/` beside `internal/gitexec/`: the git and tmux version floor and its version-line parser, a stdlib-only leaf.
- `docs/shared-libs/README.md`'s implementation-only libraries list gains `internal/toolfloor` — the git and tmux version floor and its version-line parser, a stdlib-only leaf — placed beside `internal/dotgit`.

**ImpactSummary:** Adds a new stdlib-only leaf package with no importer yet; the only edits outside it are one PATTERN background section, one overview tree line and one shared-libs list line.
