```yaml
lens: fix-proposal
outcome: reported
```

No findings against the proposals the fixer carried out as proposed. Two LOW notes on code the fix touched, then the tool-call lines.

**LOW — F1 (own change, but the base-error handling affects the board way forward):** `internal/configsync/configsync.go:1046-1048`, the new `FileError` on the template base.
- `hubreconcile.worktreeErrorFor` copies `FileError.Path` into `WorktreeError.File`, and `WorktreeError.Error` then appends "fix <base path> (or run "lym config profiles --layer default" ...)" (`internal/hubreconcile/hubreconcile.go:101`).
- For a corrupt base, that second suggestion reruns the same failing reconcile. The first suggestion and the cause text ("delete it") do name the right file, so the operator is not misled.
- It is not a regression.

**LOW — F9 (fixed as proposed, leaves one twin stale):** `docs/overview.md:378` still says "prunes but never fills a present one" and then adds that a seed-only file follows its template.
- The fixer corrected `internal/configcli/hubwide.go:163` and `internal/configcli/doc.go:92`, which the review named.
- The `docs/overview.md` sentence is the same wording for the same command, and it still reads as a contradiction.
- Its own tail clause does carry the seed-only exception, so this is mild.

Proposals I tried to break and could not:
- **F2:** I tried a base committed as stale and then rewritten to current.
  - A stale base committed through `git commit -- <base>` makes the apply's final bytes differ from HEAD, so the base lands in the config commit.
  - The `wantShow` order (lenses, loom, `profiles.template-base`, tapestry) is path-sorted and the tier 1 run passed.
- **F4:** The no-mail rerun holds because the first walk rewrites the base to the template. The solid `code_max_bounces: 5` then has base equal to template, so it produces no `differs` line on the second walk.
- **F6:** The restore row works whether or not the forged board tracked a base, because `--ignore-unmatch` plus `--allow-empty` keeps it valid. It covers removal of a written base only, not restore of prior base bytes. The finding allowed "restored ... or removed".
- **F5, F8:** No breakage found.

Tool calls (1 of 6):
- `grep -rn "never fills\|never fill\|FollowTemplate("` over the worktree's Go, md and yaml files: to find stale twins of the F9 wording and other `FollowTemplate` callers. My context did not show `docs/` or the other callers.
