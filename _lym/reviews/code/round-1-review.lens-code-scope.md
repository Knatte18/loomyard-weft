```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each diff change to the Goal points in the decision record, which has no `Added after Discussion` entries.

**Per change:**
- **C1. The line decision in `rememberRuns`.** `landnotice.go:118` changes to `if note.Requested`, and the `members` parameter is gone. This traces to Goal §1, bullet 1: a member's line goes out only when its reading is on, and the follow-up stays on that line.
- **C2. `LandNoticeState.Member` removed.** This traces to Goal §1, bullet 2, which says `Member` goes if nothing else reads it. A grep finds no other reader.
- **C3. The board membership read removed.** This covers `landNoticeTasks`, `isBeamMember`, the `boardengine` import and the `landNoticeBoard` episode in `loop.go`. C1 and C2 force it, because nothing else used them.
- **C4. Doc wording drops the exception.** This covers the `doc.go` package bullet, the `landnotice.go` header, and the comments of `landNotices`, `rememberRuns`, `pullRequestFollowUp`, `Deps.PullRequestRecorded` and the daemon field. Goal §1, bullet 2 names the first three. C1 forces the others, since their text became stale.
- **C5. The passage in `contracts/orchrole/role-beams.md:55-56`.** C1 forces it: the old wording stated the exception that C1 removes.
- **C6. `TestLandNotices` rows and harness.** This includes the two later commits that put the entry on the board for beam rows. C1 forces it.

**Per decision:**
- **§1 bullet 1 (`rememberRuns` sends a line only when the reading is on, with the follow-up kept):** built by C1.
- **§1 bullet 2 (docs and `Member` drop the exception):** built by C2 and C4.
- **§1 bullet 3 (`PATTERN-landing.md` and `PATTERN-config-layers.md` lose any mention):** needs no change. A grep finds no exception wording; the one `land_notice` hit, `PATTERN-config-layers.md:119`, is about the lookup.
- **§1 bullet 4 (note `beam-pr-describes-itself`):** already met. The board holds that entry (T3) and it depends on this task. The diff starts no work on it.
- **§2 F26:** already met at the merge base by `118bee8ad`, which is an ancestor of HEAD. `hubreconcile/reinterpret.go:104-123` gives the told `CodeWorktrees` to `CarryBeamTaskSettings`.
- **§2 F2:** already met at the base. `config.go:78`, `configreg.go:147` and `mergeparent.go:86` all use `beamsetting.Module`, and a grep finds no `"beam"` literal at those sites.
- **§2 F27:** already met at the base: `configreg_test.go:28` carries the `//testtiming:keep` marker with its reason.
- **§2 F28:** already met at the base. I checked only the case names in `runsettings_test.go`, and no name repeats.
- **§2 F5:** already met at the base: `hubreconcile/reinterpret.go:112-116` guards the carry with a per-machine marker beside the build stamp.
- **§2 F13:** already met at the base: `boardengine/store.go:291-296` returns early for a project, so its retired keys stay in `Unknown` across a save.

**Findings:** none. Every change traces to a Goal point or is forced by a traced change. The six §2 findings are already fixed in the tree at the merge base, so the Goal asks for no further change there. `_lym/landing/summary.md` (Design decisions) says the same.

Tool calls:
1. `sed` of `boardengine/store.go:290-335`, `hubreconcile/reinterpret.go:105-125`, and `git show 118bee8ad --stat`. My context lacked the F13 project fold, the F5 per-machine marker, and proof that `118bee8ad` changed the §2 files.
2. `lym board list`, to check that the board entry `beam-pr-describes-itself` exists. The earlier `lym board get` had failed.
