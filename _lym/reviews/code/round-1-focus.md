---
round: 1
focus:
  - "internal/runpusher/run.go and start.go: do the D3 start conditions, the exit-then-reread step and the lock handoff leave a commit made during exit unpushed, or let two pushers run for one worktree?"
  - "internal/runpusher/run.go and internal/fabricengine/pushanchored.go: does the owed backoff (1 s doubling to 60 s, one creel notice at 60 s) hold neither the push lock nor the slug teardown lock while it sleeps, and write no owed-class failed-push record?"
  - "internal/fabricengine/pushclassified.go, takeover.go, handover.go and replay.go: on a prime rejected push, does the takeover read run before any replay, does the archive push of the prime tip precede the hand-over mark, and does the replay drop only the handed-over runs' commits while a failed read, archive or hand-over replays nothing and retries?"
  - "internal/fabricengine/pusherreads.go: do the ahead reads (no-upstream branch, pruned tracking ref, Warp-SHA code tip, commits no transition names) match the D5 dobby-tick rules, and does `RefusedPushStands` count only the non-owed classes?"
  - "internal/statuscommit/statuscommit.go, internal/battencli/commitstatus.go and internal/loomcli/wiring.go: does every transition (running, skipped, start row, seeded Worktree-Create) commit and return with no push, fetch, reconcile or `pauserecord.Admit`, and is the start-row exception and the marker's push_pending forcing fully retired?"
  - "internal/loomcli/pusher.go and prepush.go: does the pusher read a covering pause before the Finalize skip, stop and exit on it, and hand over an entry owned elsewhere with `InsidePair` true; do `commit-records` and `commitRunRecords` commit without a pre-push check and start the pusher?"
  - "internal/battenshed/teardown.go: is the push made with the prime lock released, is the removal gated on origin's tracking refs holding the local tips with a loop back to push when a tip moved, and does a refused push halt the teardown with the pair kept?"
  - "internal/pairteardown/runread.go and internal/runproc (`EndSubject`): does `CodeTipMissing` stay unset only when the pair stands here and holds the tip, and does the pusher register under `runproc.HubOwner` with `RolePush` so a run's end spares it?"
  - "internal/beamcli/dobbydeps.go and internal/battencli/continue_pass.go resume paths: does the dobby tick and a resume start the pusher under the D3 start conditions, and does the resume start it before the run acts?"
  - "pattern/PATTERN-run-fsm.md, PATTERN-pair-remote.md, PATTERN-owner-machine.md, glossary.md: does each declared PATTERN change land with its declared text, and does any remaining wording still state a transition push before the run acts or the start-row exception?"
  - "Test changes: do deleted or rewritten tests in statuscommit, loomcli and battenshed loosen an assertion that exists at the review base without a `Loosened-Check:` trailer, and does the slackengine relay_test change carry one?"
---

Seed focus for round 1: the diff is large and spans the pusher lifecycle, the fabric push and takeover paths, the transition seams and teardown.
Entries name where the decisions D1, D3, D5, D6 and D7 carry their invariants.
