```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every member of the sets its findings rest on.

**Left unchecked:** F2, F4, F6 and F7, none of which rests on a set. F1, F3/F9, F5 and F8 were checked.

**Findings**

1. **LOW.** `cloneCmd.Flags().Bool("reset", ...)` help in `internal/fabriccli/fabric.go` (F5's set of "`--reset` is idempotent" claims).
   - The flag help still reads `remove an existing hub before cloning (idempotent re-clone)`.
   - A finished clone leaves its module-config commit unpushed, so repeating the clone with plain `--reset` is refused. That is the reason the fix gave when it rewrote the S6 step in `SANDBOX-CORE-SUITE.md`.
   - The neighbouring `Long` text was edited and this line was left as it was.
2. **LOW.** The `written` list for the Bolt checkout in `hubWorkOriginLacks` (F1).
   - It skips `board.lock`, `board.push.lock` and the render manifest, but not `.gitrepo-push.lock` or `*.swaplock`.
   - This machine's live `_bolt` holds `.gitrepo-push.lock` and `board.json.swaplock` as ignored files, so a plain `--reset` of such a hub still names them as work.
   - No current code writes any of these at the root, `board.lock` and `board.push.lock` included: all locks now go through `lock.TrackedLockPath` into `.lym/locks`. The line drawn between the two groups of leftovers therefore does not follow from "what lym writes itself", which the new `doc.go` text states.
   - The review's criterion allows leaving the other leftovers out, so the fixer-report should say this plainly. No code change is required.

**No gaps found in the other sets**

- **F1, prime store:**
  - The other exclude writers cover junction names and `.scratch`, which are links, and the warp exclude list, which is skipped.
  - `.vscode` is covered by the two files `WriteConfig` writes.
  - The heddle outputs I traced (target files and `.heddle`) are gated on the exclude list.
- **F1, records store:** the only excludes written are `.weft/` and `.lym/`, both skipped.
- **F1, statements of the rule:** `fabric.go` help, both `doc.go` files, the `hubWorkOriginLacks` comment and `SANDBOX-FABRIC-SUITE.md` are updated. The task summary under `_lym` still says "Every ignored path counts as work". The fixer left it unedited on purpose, and the report needs to say so.
- **F8:** the places that name a `--reset` clone are `existingHubError`, `RefuseHubWithoutBolt` and the no-upstream push refusal in `pushclassified.go`, which already names `--discard-work`. `docs/getting-started.md` only lists the flag.
- **F3/F9:** the store reads the review named are covered. `linkedWorktreeNames` still treats any `Lstat` error as "not a worktree", but that code predates this fix and is outside the change set.

**Tool calls (4)**

- grep for exclude and `.gitignore` writers: my context lacked the full list of paths lym excludes.
- grep for `--reset` across docs and code: my context lacked the other places that name or promise it.
- read of `scratchlink.go` and `worktreeseam.go`, plus a grep for warp-exclude users: my context lacked what `.scratch` and the heddle render write.
- grep for tool directories under worktree roots: my context lacked any other writer of ignored files in a worktree.
