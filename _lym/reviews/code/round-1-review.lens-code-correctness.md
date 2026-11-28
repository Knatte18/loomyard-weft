```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I ran each function and branch the diff adds or changes on a concrete input and checked the result.

**Per function or branch: the case I ran, and whether it is correct**
- `CloneHub` with `DiscardWork` set and `Reset` unset (`internal/fabricengine/clone.go:341`): refused before any I/O, and the text names the way forward. Correct.
- `CloneHub`, the early `checkResetTarget` call in the two-argument and one-repository forms, then the late `resetHub` call: both get the same `opts`, so they decide the same way. The one-argument form runs only the late check. Correct.
- `checkResetTarget` with `DiscardWork` set over a hub holding only store work: `lost` is `work.inUse`, which is empty, so the hub is removed. With a pair present, `lost` is the pair and the way forward is the in-use text. Correct.
- `checkResetTarget` when `hubWorkOriginLacks` fails, for example a Bolt `.git` file whose gitdir no longer exists: it returns `reset: read the status of …` with no way forward, before `DiscardWork` is looked at. Not correct (finding F3).
- `hubWork.all` with both lists empty, and with only `inUse` set: it returns a fresh slice and never aliases `inUse`. Correct.
- `hubWorkWayForward` with store work only: it copies `opts`, sets `DiscardWork`, and `cloneResetCommand` adds `--into <cwd> --reset --discard-work`. On the non-reset path in `existingHubError` it adds `--reset` too. Correct.
- `hubWorkWayForward` with `inUse` set: it names land, remove or move aside, and offers no discard. Correct.
- `existingHubError` on an unfinished hub with store work: it names the work and the discard clone. On a read error it names the move aside. Correct.
- `cloneResetCommand` with `DiscardWork` set and unset: the flag is appended only when set. Correct.
- `hubWorkOriginLacks` stores loop: it reads the code store, the records store and Bolt (with `ownsRefs` false). Bolt's stash is covered, because `refs/stash` is a shared ref the records store reads. Bolt's own HEAD is not covered (finding F2).
- `storeWorkOriginLacks` when `.git` is missing: returns nil. When `Lstat` fails for another reason, such as EACCES, it also returns nil silently (finding F4).
- `storeWorkOriginLacks` cap with 0, 5 and 6 ignored paths: 0 adds no item, 5 names all five with no "more", 6 names five and adds " and 1 more". Correct.
- `ignoredWork` on the live hub's `_bolt` (`.board-rendered.json`, `board.lock`, `board.push.lock`, `.lym/`): `.lym/` is skipped and the other three are counted. In the live prime, `_lym`, `.lym` and `.scratch` are skipped as links, while `CLAUDE.local.md`, `AGENTS.override.md`, `.claude/` and `.vscode/` are counted. Not correct (finding F1).
- `ignoredWork` with a nested `.lym` entry such as `internal/loomshed/.lym/`: `path.Base` is `.lym`, so it is skipped. When `fslink.IsLink` fails, the entry is counted, which errs the safe way. Correct.
- `countCommitsOriginLacks` on a one-repository records store: the fetch refspec maps `refs/lym/heads/*` to `refs/remotes/origin/*`, so `--remotes` covers origin's branches. Namespaced tags land in `refs/tags`, and `ls-remote` covers what origin advertises. Correct.
- `countCommitsOriginLacks` on a two-repository records store with the clone's unpushed module-config commit: counted, as the code intends. Correct.
- `countCommitsOriginLacks` on an unborn records store, or one with no origin: `--ignore-missing` gives 0. When `ls-remote` fails it keeps the count, which refuses rather than loses work. Correct.
- `resetHub` and `removePath`: unchanged apart from the `dirtinessNA` text. Correct.
- `refuseUnseededBoard` message: the format arguments line up with the `%s` verbs (`refused`, `RecordsNamespaceRoot`, `DiscardWorkFlag`). Correct.
- `runCloneWithReset` and the `cloneFlags` struct: every flag is passed through, including `discardWork`. Correct.

**Findings**

F1 — BLOCKING, design — `internal/fabricengine/clone.go:1078` `ignoredWork`
- **Summary:** Files lym itself writes count as the operator's work, so a plain `--reset` of any hub that has been used is refused.
- **Bolt checkout:** the counted files are `.board-rendered.json` (`internal/boardengine/render.go:22`), `board.lock` and `board.push.lock` (`internal/fabriclayout/hubpaths.go:16`, `:20`).
- **Prime:** the counted files are heddle's rendered `CLAUDE.local.md` and `AGENTS.override.md`, `.heddle` and `.claude/settings.local.json`. All four come from fabric's own `warp_exclude` list (`internal/fabricengine/template.yaml:3`), and `.vscode/` is also counted.
- **Contradiction:** the doc comment says "every ignored path counts but lym's own", but only links, `.lym` and `.weft` are skipped.
- **Evidence:** `git ls-files --others --ignored --directory` on `/home/knatte/Code/loomyard-LYMHUB/_bolt` and `/home/knatte/Code/loomyard-LYMHUB/loomyard` listed these files, and `stat` showed the prime's two rendered files are regular files, not links.

F2 — MEDIUM, design — `internal/fabricengine/clone.go:1011` (Bolt store read with `ownsRefs: false`)
- **Summary:** A commit reachable only from Bolt's own HEAD is removed by a plain `--reset` and never named. This covers a detached HEAD, or commits made during a paused rebase.
- **Why:** `countCommitsOriginLacks` runs in the records store, where `HEAD` is that store's HEAD, not Bolt's. `--branches` does not cover a detached worktree HEAD, and per-worktree refs (`refs/worktree/*`, `refs/bisect/*`) are not read either.
- **Evidence:** `clone.go:1054-1056` (Bolt returns before the commit count) and `clone.go:1107` (`rev-list … HEAD --branches --tags`).

F3 — MEDIUM, design — `internal/fabricengine/clone.go:944-947` `checkResetTarget`
- **Summary:** When any store's git read fails, `--reset` is refused with no way forward, and `--discard-work` cannot get past it. Examples are a Bolt `.git` file pointing at a gitdir that is gone, or a corrupt records store.
- **Why:** the read error returns before `opts.DiscardWork` is consulted.
- **Regression:** the code used to read only the code store, so a hub with a broken Bolt or records store could be reset. Now the operator's only move is removing the hub by hand, and the message does not say so.
- **Evidence:** `clone.go:944-947`, and `clone.go:1032-1035` (status error).

F4 — LOW, design — `internal/fabricengine/clone.go:1027` `storeWorkOriginLacks`
- **Summary:** Any `Lstat` error on `<store>/.git`, such as permission denied or an I/O error, is treated as "no repository", so that store's work is removed without being named.
- **What it should do:** only `os.ErrNotExist` should mean no repository; any other error should go back to the caller.
- **Evidence:** `clone.go:1027-1029`.

Tool calls: none; the inherited context held everything this lens needed.
