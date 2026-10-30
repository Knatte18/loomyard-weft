---
round: 1
focus:
  - "internal/landingshed PR-Review wait loop (D9, cards 13 and 14): does each merged, closed and approved outcome match the decision record's state table, including stopping a live session before the re-read, resuming only a session it stopped, and taking no parent lock on any path?"
  - "PR-Review covering pause (Added after Discussion): does the row stop its session and hold in process until the pause lifts, with no transient mark and no gate, push or GitHub call while covered, and is the wake interval bounded to one second to one minute with GitHub polled at most once a minute?"
  - "D8 agent-commit gate: is a verdict counted only when head and tree are unchanged across the gate, is the push made only while the head still equals the gated head, and is the failure mail sent once per failing head?"
  - "Finalize through the pull request (D11, card 15): does the resolver run first under the parent lock, are the no, yes-without-record and merged-by-someone-else paths right, is the expected head passed to the GitHub merge, and does every refusal halt behind LandingStopPrefix with a way forward?"
  - "Pull-request mode resolver (D2 to D4, cards 2, 3, 8): are all nine parent and field combinations right, is a missing GitHub fact a no only when the mode came from the parent alone and a halt otherwise, and does a beam landing read its project entry against the parent's beam.yaml?"
  - "Pair teardown close (D12, card 16): is the record read and the pull request closed with a comment before removal, does a failed close or gone worktree never stop the teardown, and does the Warn name the task branch?"
  - "Session start and publish guard (D7, cards 5, 6, 14): does the unawaited start carry the prompt and register in runproc, is resume by session id used for a stopped session, and does the DenyPublish switch install the git push and gh denies for this session alone?"
  - "Retired keys and recipe wiring (D3, D13, cards 4, 12): do loom.pr_review and darn.pr_review load as retired and prune, is pr_account absent everywhere per the Added after Discussion decisions, and does each recipe wire PR-Review before Finalize with no entry_gate and no on_stuck?"
  - "Declared PATTERN changes: does the diff land each entry in the record's PATTERN changes section with its declared text, with PATTERN-github-auth carrying no named-account clause?"
  - "Quiet check (D10, card 17): does readAgentActivity count a child status running at PR-Review as a live wait, and does the wait mark get set, re-set after each gate and cleared on every way out?"
  - "Test economy and the change description: do new tests duplicate an existing test's coverage, and does every claim in _lym/landing/summary.md bear out against the diff?"
---

Seed focus from the decision record and plan overview; the plan's later cards, the diff itself and the commit trailers are read in the round.
