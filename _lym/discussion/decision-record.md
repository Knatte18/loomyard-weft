# Decision record — pattern-spawn-and-links-scans

## Goal

Two rules that stand today only in the loomyard `CLAUDE.md` become PATTERN entries, each held by a scan, so they survive that file's deletion: every agent lym spawns runs as an interactive tmux session driven through reed, never headless `claude -p`, and every filesystem link goes through `internal/fslink`.

## Scope

- A PATTERN entry for spawning an agent, with a background file and a scan.
- A PATTERN entry for creating a filesystem link, with a background file and a scan.
- Both scans live in `cmd/lym` beside the other module-wide scans, each with fixture cases of a violation found and of a clean tree passing.
- Out of scope: any change to `CLAUDE.md`, which the follow-up task `claude-md-lym-rules-out` deletes, and any claim about Windows junction behaviour beyond what the scan holds.

## Decisions

### D1 — Spawn entry under "Agents and prompts"

`PATTERN-interactive-spawn` joins the "Agents and prompts" group: spawning an agent runs it as an interactive tmux session driven through reed, never headless with `-p` or `--print`.
It carries `(test)` and a background link.
Rationale: the rule is about how an agent is launched, which is that group's concern; the economic reason (subscription coverage) goes in the background file, not the entry line.

### D2 — Spawn scan: three static shapes in production Go files

`cmd/lym/interactivespawn_test.go` (new) walks every production Go file of the module, `tools/` included, parses it, and reports:

- a call with a `-p`, `--print` or `--print=…` string-literal argument and another argument naming claude: a string literal whose base name is `claude` or `claude.exe`, or an identifier, selector or call whose name holds `claude` in any case;
- a string literal whose whitespace tokens, quote characters trimmed, hold a token naming claude followed later by a print-flag token;
- any string literal in `internal/shuttleengine/claudeengine/` holding a print-flag token, since that engine builds claude's launch lines from flag fragments.

Comments are never read, so the docs that name `claude -p` pass.
An allowlist keyed `<file>` with a reason admits a site, and reports its stale entries.
Bound: the scan sees static shapes only; a flag built at runtime, or a line assembled from fragments outside the claude engine's directory, passes. Test files are not scanned, so a test may hold the forbidden text as data.

### D3 — Links entry under "Paths and geometry"

`PATTERN-fs-links` joins the "Paths and geometry" group: creating a filesystem link goes only through `internal/fslink`'s `CreateDirLink`, with no `os.Symlink` or `os.Link` call outside `internal/fslink`.
It carries `(test)` and a background link.
Rationale: the entry states only what the scan holds; whether a Windows junction behaves as fslink's documentation says stays unsettled, as `internal/fabricengine/doc.go` records, and the background file points there.

### D4 — Links scan: production files, one allowlisted site

`cmd/lym/fslinkonly_test.go` (new) walks every production Go file of the module, parses it, and reports each `os.Symlink` or `os.Link` call, through any name the file binds `os` to, outside `internal/fslink/`.
`tools/deploy/main.go`'s `copyEntry` copies a symlink from the plugin tree verbatim, which may point at a file, and fslink has no file-link primitive; it is admitted by a `<file>:<call>` allowlist entry with that reason.
Bound: the entry lets through that one deploy-tool site, and test files are not scanned, because tests plant links as hostile or foreign fixtures that fslink's refusals would block.

### D5 — Docs

The two background files under `pattern/` carry each scan's scope and bounds.
No package `doc.go` changes: neither scan changes a package's behaviour, and `internal/shuttleengine/doc.go` already states the spawn rule.
Rationale: CLAUDE.md asks for PATTERN in the same commit as a new invariant, which D1 and D3 meet.

## Constraints

- `PATTERN-test-speed`: each scan file carries one `tier0` tag, and its tests run in parallel unless they touch shared state.
- `PATTERN-test-isolation`: the scans use the shared `internal/testkit/scankit` harness.
- The overview stays under `pattern.MaxOverviewBytes`, and each new entry's rule text under `pattern.MaxEntryTextChars`.
- `CLAUDE.md` is not touched.

## Auto-mode assumptions

- No operator is present; the entry names, group placement and scan file names are my own picks, recorded in the support log's question ledger.

## Open risks

- The spawn scan's identifier heuristic (a name holding `claude`) could flag an unrelated call that takes a `-p` literal; an allowlist entry with a reason is the remedy.
- A future claude-engine string carrying a `-p` token for another tool would need an allowlist entry.

## Acceptance criteria

- D1: `PATTERN.md` carries `PATTERN-interactive-spawn` in "Agents and prompts" with `(test)` and a background link, and `pattern.Check` passes on the real tree.
- D2: the spawn scan passes on the current tree, and its fixture cases find each of the three shapes and pass a clean tree, a comment naming `claude -p`, and an unrelated `-p` call such as tmux's.
- D3: `PATTERN.md` carries `PATTERN-fs-links` in "Paths and geometry" with `(test)` and a background link, and states no Windows claim.
- D4: the links scan passes on the current tree with the one deploy-tool allowlist entry, and its fixture cases find `os.Symlink` and `os.Link`, an aliased `os` import, and pass a call inside `internal/fslink`, a comment, and a test file.
- D5: both background files exist and name their scan, and `CLAUDE.md` is unchanged.

## PATTERN changes

- `PATTERN-interactive-spawn` — adds: Spawning an agent: an interactive tmux session driven through reed, never headless `claude` with `-p` or `--print`. (test) — [background](pattern/PATTERN-interactive-spawn.md)
- `PATTERN-fs-links` — adds: Creating a filesystem link: only `internal/fslink`'s `CreateDirLink`; no `os.Symlink` or `os.Link` outside `internal/fslink`. (test) — [background](pattern/PATTERN-fs-links.md)
