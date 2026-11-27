```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which maps the diff's changes to the amended board entry's decisions.

**Decisions**
- G1, Goal Idea (a): give testkit a recorded reed and creel session stand-in usable at tier0. The reed half is built by C1–C6. The creel half is deferred by A1, which wins where it differs.
- G2, Goal Idea (b): move the PR-Review session tests onto that stand-in. Built by C7.
- A1, Added after Discussion (parent, 2026-11-27), option B:
  - (i) a tmux-exec and pane-age seam on `reedengine.New`: C1, with C2–C4 forced by it;
  - (ii) `internal/testkit/tmuxrec` with recorded transcripts and their source lines, a tier0 replay test, and a tier2 recorder and contract test: C6;
  - (iii) move `sessionLiveness` and `sessionRegistrar` from `loomcli` into `reedengine`: C5;
  - (iv) PR-Review session tests over the real reed and runner at tier0: C7;
  - (v) leave the `MailAgent` closures and envkit unchanged: honoured. The diff stat touches no envkit file, and the `mails` and `MailAgent` seams in `prreview_wait_test.go` are untouched.

**Changes**
- **C1** `reedengine.Outside` and `NewOver` (`internal/reedengine/lock.go:85-145`): traces to A1(i).
  - The `Tmux`, `TmuxVersion` and `PaneAge` fields are the requested seam.
  - The `CapMemory` and `NewGUID` fields go beyond "tmux-exec and pane-age", but A1(iv) forces them:
    - tier0 pane creation would otherwise reach the user bus (`panememory.go:46`);
    - the replay must mint the guids its recorded args name, such as `.../launch/guid-1.sh` in the recordings.
  - Forced, so no finding.
- **C2** told-tmux spawn branch (`internal/reedengine/lifecycle.go:472-479`): forced by C1. A told tmux would otherwise still exec the real binary for the server spawn.
- **C3** `paneProcessAge` exported as `PaneProcessAge` in `proctree_linux.go` and `proctree_windows.go`, with test renames in `proctree_test.go` and `ownwindow_integration_test.go`: forced by C6, since the recorder passes reads through to it (`record.go:107`).
- **C4** `reedengine/doc.go` two lines naming `New` and `NewOver`: forced doc for C1.
- **C5** `sessionreads.go` and `sessionreads_test.go` added, removed from `loomcli/landingdeps.go` and `landingdeps_test.go`, and rewired in `sharedbootstrap.go:439-447`: traces to A1(iii).
- **C6** `internal/testkit/tmuxrec`: `tmuxrec.go`, `record.go`, `session.go`, `recordings_test.go`, `testmain_test.go` and `testdata/*.json`. Traces to A1(ii) and G1.
- **C7** `landingshed/prreview_session_test.go`, `prreview_test.go` and `prreview_wait_test.go`: traces to A1(iv) and G2.
- **C8** `pattern/PATTERN-test-stand-ins.md:67-68`, a paragraph naming `tmuxrec` as the stand-in below `reedengine`: traces to no decision.
  - No decision record entry asks for a PATTERN change, and this round's directive declares no PATTERN changes.
  - The paragraph is not forced: no existing text of the background is left stale by the kit's arrival.
  - No `Pattern-Departure:` trailer explains it (`git log` over the range).

**Findings**
1. **LOW, scope.** Location: `pattern/PATTERN-test-stand-ins.md:67`.
   - Summary: the diff adds a rule-document paragraph that no decision or declared PATTERN change licenses, and no traced change forces.
   - Evidence: the diff piece `010-pattern.diff` lines 9-11; the decision record's `## Decisions`, `_lym/discussion/decision-record.md:21-25`; the commit log of the range, which carries no trailer.
   - Fix: drop the paragraph, or have the parent declare it.

No other unrequested change, and no unbuilt decision.

I made no tool calls; I worked entirely from the context I inherited.
