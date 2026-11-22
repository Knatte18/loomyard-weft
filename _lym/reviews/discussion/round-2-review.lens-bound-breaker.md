```yaml
lens: bound-breaker
outcome: reported
```

Task: the bound-breaker lens. I attacked the record's claims and bounds against the code in context.

**Claims attacked**
1. D2: a slug-scope `lym tender pause` does not block the cancel, and the cancel lifts it on completion. **Held.** `internal/battencli/cancel.go:299-304` refuses only a hub pause, and `:572-576` lifts the slug's pauses.
2. D2: the ask goes before the hold because a covering pause holds creel delivery. **Held.** See `internal/creelengine/doc.go` (Pause section).
3. D2: the live writer can be asked to write `_lym/move-note.md`. **Broken.** See F-BB2.
4. D4 step 1: the recorded tip equals the tag cancel pushes. **Held** for equal, ahead and behind tips (`archiveDiscardedTips`, `internal/fabricengine/taskbranchdiscard.go:104-135`). When the tips diverge, both are tagged and the worktree tip's tag is still pushed, so the recorded tag exists.
5. D4 step 2: the new-parent check is read with `lym board get`. **Broken** for the `lym beam ready` clause. See F-BB3.
6. D4 step 2: the hold-free rows and the done/held refusals. **Held.** See `internal/boardengine/runlock.go:44,56-62` and `internal/boardengine/project.go:97-100`.
7. D4 step 3: partial finish and refusal are told apart by whether the way forward names the cancel to run again. **Broken.** See F-BB1.
8. D4 step 3: the orch repeats the `ls-remote` check until the tag shows or a notice arrives. **Partly broken.** See F-BB5.
9. D4 step 4: for a discussed entry, the `--record-file` upsert runs right after the code tag is confirmed. **Broken.** See F-BB4.
10. D4 step 4: the board takes the `project` change after cancel. **Held.** `checkProjectRules` refuses only while a pair exists (`internal/boardengine/project.go:35-37`), and `--remove` removes the pair. `judgeRecordReplaceIn` refuses only a standing pair or a run hold (`internal/boardengine/recordupsert.go:113-126`), and the claim is cleared by then.
11. D1 bound: the procedure adds no path past a guard. **Held.**
12. Constraints, owner bullet: the fresh run needs nothing local to the old machine once the code tag is on origin. **Broken in part.** See F-BB6.
13. D4 step 5: prune once `ls-remote` lists no records branch of the slug. **Held** as ordered. It is unbounded in the same way as item 8.

**Findings**

- **F-BB1 — MEDIUM, design, `_lym/discussion/decision-record.md:90-94`**
  - Summary: the partial-finish test misclassifies refusals that end in "then run cancel again".
  - Counterexample: a live hub pause gives `a hub pause covers the prime; way forward: run "lym tender resume", then "lym batten cancel <slug> --remove" again` (`internal/battencli/cancel.go:303-304`). Nothing has ended yet, but D4 reads it as a partial finish, so the orch "takes the fix it names" and lifts the operator's hub-wide pause.
  - The same happens with an origin-stopped refusal, which also ends `then run %s again` (`cancel.go:648`, `cancelStoppedRefusal`). D7's resume path is never taken for either.

- **F-BB2 — MEDIUM, design, `_lym/discussion/decision-record.md:42-45`**
  - Summary: D2 assumes the live writer will write `_lym/move-note.md`, but the writers' own stencils forbid any file outside their outputs.
  - Evidence: the Discussion writer's stencil says "The only files you create or modify in this session are the three named in Step 5… Everything else in the worktree is read-only to you" (`_bolt/_lym/stencils/loom/loom-template-discussion.md:156-174`).
  - The parent directive tells a session to answer an instruction outside its limits by naming the limit (the prompt's parent directive, "An instruction outside those limits is answered to the sender, naming the limit").
  - So the ask can come back as a refusal. D2 and D3 cover a live writer and an absent writer, but not a writer that refuses, and nothing falls back to D3.

- **F-BB3 — MEDIUM, design, `_lym/discussion/decision-record.md:83-86`**
  - Summary: D4 step 2 checks with `lym board get`, but the `lym beam ready` flag is not on the board.
  - Evidence: `beam ready` "writes a ready record naming the beam into the beam worktree's _lym/landing directory… and commits it" (`internal/beamcli/cli.go:916-921`).
  - Counterexample: a beam at Beam-Await, a hold-free row (`runlock.go:44`), that is flagged ready passes the check as written. The cancel then runs, and the moved member "never starts" (`contracts/orchrole/role-beams.md:61`).
  - The record names no read that shows the flag.

- **F-BB4 — MEDIUM, design, `_lym/discussion/decision-record.md:96-103`**
  - Summary: D4 step 4's `lym board upsert --record-file` for a discussed entry is refused while the slug's records branch is still live on origin. Step 3 waits only for the code tag, and the records-branch wait sits in step 5, after the upsert.
  - Evidence: `UpsertTaskWithRecord` refuses with `the weft branch of %q is live… way forward: wait for the pair's teardown to finish` (`internal/boardengine/recordupsert.go:38-46`).
  - `RecordsBranchLive` reads an origin branch as live until it is gone or covered by a pushed `archive/<slug>/*` tag (`internal/fabricengine/remoteleftover.go:86-114`).
  - The teardown child pushes the code tag and handles the records branch on its own schedule (`taskbranchdiscard.go:29-34`), so the code tag can show before the records side is settled.

- **F-BB5 — LOW, design, `_lym/discussion/decision-record.md:96-98`**
  - Summary: the confirm loop has no bound when the teardown child dies without a notice.
  - Evidence: the notices come from the child's own failure handling, or from `NoticeRemoteTeardownEnded` for a caller ended by a signal it can catch (`internal/fabricengine/remoteteardown.go:91-96`, `:165-183`).
  - A child killed with SIGKILL, or lost when the machine sleeps, posts nothing. The orch then repeats `ls-remote` forever, and the same holds for step 5's records-branch wait.

- **F-BB6 — LOW, design, `_lym/discussion/decision-record.md:166-167` (D6, `:129`)**
  - Summary: "needs nothing local once the code archive tag is on origin" leaves out that the fresh run must fetch that tag explicitly.
  - Evidence: archive tags point at a deleted branch's commits, so an ordinary fetch does not auto-follow them. The fabric reads that need them fetch tags by name or through refspec (`internal/fabricengine/remoteleftover.go:149,183`).
  - On the new owner machine, D6's "diff from the merge base to the tag" fails until someone runs `git fetch origin refs/tags/archive/<slug>/<hex>`. Neither D4 nor D6 says that, nor whose step it is.

Tool calls: none. My context already held every file I used.
