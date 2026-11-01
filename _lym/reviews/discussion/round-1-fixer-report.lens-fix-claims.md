```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. Every bound, behaviour and code fact the fix added or rewrote, checked against HEAD and the Claude Code memory docs.

Claims that hold:
- `BoardWriteLockPath` joins `BoardDir(hub)` (`internal/fabricengine/junctionnames.go:92`); `setAsideTag`'s empty-slug fallback is `board` (`internal/fabricengine/replay.go:483-491`); the geometry test's policed-token list spells `_board` (`internal/lymcwd/enforcement_test.go:209`); `runproc.HubOwner` is `const HubOwner = "_board"` (`internal/runproc/record.go:21`).
- `lymcwd` imports only stdlib, `dotgit` and `weftname`; neither of those imports `runproc` in production (their only internal imports are `weftname` and test kits), so "lymcwd imports neither runproc nor any package that does" holds.
- "takes the exclude lock as D5's seeding does": `seedGitExclude` goes through `mutateGitExclude`, which works under the exclude lock (`internal/fabricengine/junction.go:559-560`).
- The Claude Code facts in D3 match the fetched memory docs: a `CLAUDE.md`, `.claude/CLAUDE.md` or `CLAUDE.local.md` in the working directory or above stops `AGENTS.md` loading under `claude-md-or-agents-md`; the import is never read twice; `pluginConfigs` is read only from `~/.claude/settings.json`, a `--settings` file or managed settings; from v2.1.277, installed 2.1.316.
- "only sessions launched with shuttle's `--settings`": consistent with `claudeengine` lines that pass no `--settings` (`internal/shuttleengine/claudeengine/command.go:270`, `:292`).

Findings:

- LOW — D7 acceptance criterion "a heddle set-aside is tagged `set-aside/replayer/heddle/<short sha>`". The criterion fixes the replayer's name as `heddle`, but D7 and D8 never name heddle's replayer; D7 says only "heddle registers one for `heddle.json`". The asserted tag rests on a name no decision states.
- LOW — D3 `claude-file` row and the sentence "A repository with its own `CLAUDE.md` already keeps its `AGENTS.md` from Claude, so the render adds no import there." The import is suppressed only by a `CLAUDE.md` at the warp root or its `.claude` directory, while the docs count a `CLAUDE.md` anywhere from the working directory up; a repository whose `CLAUDE.md` sits at the anchor below the root gets the import, so its session reads an `AGENTS.md` the team's default setup leaves out, contrary to the sentence's rationale.

Tool calls beyond context:
1. Grep of `weftname`/`dotgit` imports and the exclude-lock lines in `seedGitExclude`: context held `lymcwd`'s import list but not its dependencies' imports, nor whether the seeding takes a lock.
2. Grep for the exclude lock in fabric and read of `seedGitExclude`'s body: the first grep showed no lock in the seeding lines, so I needed `mutateGitExclude`'s lock statement.
