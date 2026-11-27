```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the decision that asks for it.

**Decisions**
- **G1.** Goal, mode 1 (`review`): an agent session sits in the task's window while the row waits. It answers comments, fixes findings and ends the review on the operator's word.
- **G2.** Goal, mode 2: the pull request is accepted automatically, with no wait (`trail`).
- **G3.** Goal, mode 3: no pull request, and the row is skipped (`off`).
- **G4.** Goal, issue comment: a sanctioned way for the orch to end the review on the operator's word.
- **A1.** First added entry: restore the session as it was before 34e64ac2e, from the removed code where it still fits; `trail` and `off` stay as they are.
- **A2.** Second added entry, with three parts:
  - A: restore the session.
  - B: orch strands may run `pr-continue` on the operator's word, and every other agent stays refused.
  - C: un-retire `pr-reviewer` and `landing.pr_reviewer` and drop their reinterpretation. The session ends the review with `pr-continue` and never approves, merges or pushes.

**Changes and what they trace to**
- **C1. Session lifecycle:** `prsession.go`, the `Call`/`decide`/`openPullRequest`/`wait`/`wake` edits, `holdWhileCovered`, `closeRecordedPullRequest`, the new `PullRequestDeps` seams and the session fields on `pullrequest.Record`. Traces to G1, A1 and A2-A.
- **C2. Gate failure mailed instead of halting:** `gateHead`, `mailFailure`, `gateRetryHint` and `MailedHead`. Traces to A1: the pre-34e64ac2e `gateHead` mailed the failure the same way (old prreview.go:856–860).
- **C3. Session prompt:** `landing-template-pr-review.md`, and its entries in `stencils.go` (registry and `roleOpeningStencils`). Traces to G1, A1 and A2-C ("never approves, merges or pushes"). Leaving out the old findings send-back and the `review_mode` block traces to A2, which replaced the parent's role with the operator's word.
- **C4. `pr-continue` admission and help:** `prContinueAdmitsStrand`, the refusal text, `AudienceRole`, the Long text and the package doc. Traces to A2-B, G4 and A2's "the session ends the review with lym loom pr-continue". Clearing `MailedHead` is forced by C2.
- **C5. Config:** `landing.pr_reviewer` is back as a live key, with its template line, `ConfigSpec` category, `LoadConfig` check and `keynames.go` entry, and `configreg_test` classifies it. Traces to A2-C.
- **C6. Agents role:** `RolePRReviewer`, the template's skills line, the deletion of `reinterpret.go` and its test, and the `configreg` `Reinterpretations` removal. Traces to A2-C.
- **C7. Wiring:** `landingdeps.go` (`Sessions`, `MailAgent`, the `runner` parameter) and `sharedbootstrap.go` (`SessionLive`, `RegisterSession`). Forced by C1.
- **C8. Docs, contracts and comments:**
  - orch-role.md: the PR-Review paragraph traces to A2-B; adding PR-Review to the sessions that ask the operator in the pane is forced by C1.
  - `role-autonomy.md` traces to A2-B.
  - The recipe comments, `docs/overview.md`, landingshed `doc.go`, the prreview doc comments, `interruptpolicy.go`, `PATTERN.md` and `pattern/PATTERN-landing.md`, the dobbyengine template comment and the loomengine retired-key comment are forced: each would be stale after C1, C2 or C4.
  - The shellguard `rows.go` comment now names PR-Review as a `DenyPublish` user. It is forced by C1, although the Sampler-Session example it replaced was still true.
- **C9. Test seam inventories:** battencli `wire_seams_test`, loomcli `wiring_seams_test` and `landingdeps_test`, envkit's `noopSessions` and seams, the shuttleparity producer `buildPRReview`, and the `&shuttleengine.Runner{}` arguments in the loomcli tests. Forced by C1, C4 and C7.
- **C10.** The `seam_enforcement_test.go` allowlist, forced by C3's imports in `prsession.go`; the `Loosened-Check:` trailer on 6c9543250 names it.
- **C11. Tests:** `prreview_session_test.go`, `prreview_test`/`prreview_wait_test` and `commitrecords_test`/`config_test`. Forced by C1, C2, C4 and C5.

**Decisions and the changes that build them**
- G1: C1, C2, C3, with C4 for the ending.
- G2: no change, and none is needed. A1 keeps `trail` as it is, and `decide` still returns `done()` on `MergeAtOnce`.
- G3: no change, and none is needed. A1 keeps `off` as it is. `closeRecordedPullRequest` now also stops a recorded session, which C1 forces.
- G4: C4 and C8 (orch-role.md, `role-autonomy.md`).
- A1: C1, C2, C3.
- A2-A: C1, C3.
- A2-B: C4, which builds it wider than licensed; see finding 1.
- A2-C: C5, C6, and C3's never-approve/merge/push wording.

**Findings**

1. **MEDIUM** · scope · `internal/loomcli/prcontinue.go:247-256` (`prContinueAdmitsStrand`)
   - A2-B licenses only orch strands, and A2 adds the run's own PR-Review session; every other agent stays refused.
   - The code admits any strand whose role matches `pr-reviewer`, whatever its slug or shortname: another run's `<short>:<other-slug>:pr-reviewer`, or a two-part `<short>:pr-reviewer`.
   - It does not check the run's slug or the record's `StrandName`. The doc comment (prcontinue.go:8, loomcli `doc.go`) and `PATTERN-landing` say "the run's PR-Review session", so the admission is wider than the answer key allows.
   - Evidence: prcontinue.go:255 calls `agentname.MatchesRole(name.Role, string(agentconfig.RolePRReviewer))` and ignores `name.Slug`; agentname.go:314-324 matches on the role alone.

There are no other scope findings. Every other change traces to a decision or is forced by a traced change. Leaving out the parent-review ask and the findings send-back is licensed: A1 restores only "where it still fits", and A2 replaced the parent's role with the operator's word.

Tool calls: none.
