```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the goal points of the board entry and to the record's `Added after Discussion` entry. Every change traces to a goal point or is forced by one, and every goal point is either built or is a read-only point whose outcome the summary records. I found no scope findings.

**Decisions** (the board entry's goal points and the one added entry)
- **G1** (#223) Take every agent out of PR-Review:
  - remove the parent's ask and re-ask;
  - remove the session and the findings path (`landing.pr_reviewer`, `lym loom pr-findings`, `loom.pr.*`);
  - the row keeps only the PR-mode mechanics;
  - rely on the cold-start reviewers, after reading the C# fan;
  - decide what happens to a commit pushed to the PR branch by hand.
- **G2** (#224) The three modes, Manual, None and Trail, with no LLM in any of them:
  - Trail: Finalize opens and merges the PR at once;
  - a beam's PR lists its member PRs (read `1cd11ee5e` first);
  - Manual waits for a human whatever the autonomy;
  - read where `beam.pull_request` is read.
- **G3** (#217) Under autonomy, PR review stays with the operator.
- **G4** Close `pr-ask-live-pane-test` with no test.
- **G5** The board README shows the three modes.
- **A1** (Added after Discussion, parent) Read the body's keys in nested form, and name keys as `&lt;module&gt;.&lt;path&gt;`.

**Changes → decision**
- C1 Mode rename `on`→`review`, `merge`→`trail`, across beamsetting, the dobbyengine constants and template, the landingshed resolver and halt texts, beamcli, and the tests and fixtures → G2. The names come from the entry title, "review, trail and off", and from the operator's `trail`.
- C2 The `beam-pull-request-modes` reinterpretation, the carry mapping, the per-task reinterpretation mapping `required`→`review`, and the configreg registration and test → forced by C1 (PATTERN-config-layers: a meaning or name change ships a reinterpretation).
- C3 The board README mode legend (`boardengine/beammodes.go`) → G5.
- C4 Removal of the PR-Review session → G1. That covers `prsession.go`, the three stencils and their registry and opening-stencil entries, the `pr-reviewer` role in agentconfig and its template, the parity producer, the envkit seams, the `sharedbootstrap` session wiring, the landingshed import allowlist, and the batten and loom wiring seam tables.
- C5 Removal of the parent's PR-Review ask → G1 and G3. That covers `prreviewask.go`, `parent-review ask`, `PostAsCaller`, the unreached-tag exception (`AnswerParentAskOrUnreached`→`AnswerParentAsk`), the `pr` key `ApplyAutonomy` forced, and the `orch-role.md` and `role-autonomy.md` edits.
- C6 Removal of `pr-findings` and the send-back → G1. That covers `prfindings.go`, the reroute edge in the three recipes and the testdata, `ReviewInput.Full`, the PR-findings review input and its rubric paragraph, and the PR-Review reroute row in `TestRouting_ProgressAt`.
- C7 Retiring `landing.pr_reviewer`, the `loom.pr` area and its flat names → G1, named in nested form per A1.
- C8 `pr-continue` made an operator verb that refuses any lym agent session → G2 (Manual waits for a human) and G3.
- C9 Finalize dropping its recorded-autonomy check on a closed PR → forced by C5 and C12, since the record's `Autonomy` field is gone.
- C10 A gate failure halts the row instead of mailing the session, with `MailedHead` dropped → forced by C4, since no session is left to mail.
- C11 `foreignHeadHalt`, the halt on a commit pushed to the PR branch by hand → G1, "decide what a run does with a commit pushed to the PR branch by hand".
- C12 `pullrequest.Record` drops its session, autonomy, ask, send-back and mailed-head fields, and `ClearReviewSession` goes → forced by C4, C5, C6 and C10.
- C13 Removal of the goto reset hook (`prreviewgoto.go`, the `gotoresume.go` change) → forced by C4 and C5.
- C14 Docs and rule text → forced by C4–C11. That covers `docs/overview.md`, the `PATTERN-landing.md` rule, the final-summary spec, the `interruptpolicy` comment, the verifygate and landingshed doc comments, and the dobbyengine template comment.
- C15 Test and helper realignment → forced by the removals above. That covers the battencli and loomcli seam tables, the `TestPostSigned` rename, the burlerengine, driftengine and shedadapters call renames, the full-generation bouncer row, the recipe tests and the envkit fixture.
- C16 Commits `e69cb88` and `b0fe182`, line breaks and one comment naming `landing.pr_reviewer` in full → forced by C7 and A1.

**Decisions → changes**
- G1 → C4, C5, C6, C7, C11, with C9, C10, C12 and C13 forced. Its "rely on cold-start reviewers" point is read-only: the summary records the `review-code` lens set of the shipped template and the C# set, which match `lenses/template.yaml:421` and `lenses/sets/csharp.yaml:49`. That point needs no change.
- G2 → C1 and C8. Trail reuses the existing merge-at-once path under its new name. The beam-lists-member-PRs point needs no change: `beam-template-describe.md:38` already lists every member PR by number, as the summary states.
- G3 → C5 and C8.
- G4 → none needed. The summary records the note closing with no test.
- G5 → C3.
- A1 → C7 and C16, plus the summary's key names.

Findings: none.

Tool calls: none.
