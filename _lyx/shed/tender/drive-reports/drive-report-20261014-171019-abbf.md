---
run: tender
state: blocked
producer: Publish (step 13 of 17)
---

# Stop report: tender

Run `tender` stopped `blocked` at Publish, after the resume.

## Refusal

The verify gate `go test -tags tmux ./...` exited 1 after merging `main`.
The output is at `/home/knatte/Code/loomyard-LYXHUB/tender/.lyx/verify/verify-7.log`.
The stop's reason, verbatim: `publish_verify failed after merging parent branch "main" (exit code 1); output: /home/knatte/Code/loomyard-LYXHUB/tender/.lyx/verify/verify-7.log; way forward: run "lyx loom goto --to Webster-Burler-Review", then "lyx loom resume", in the task worktree; the Webster-Review round reads the Publish failure record and its gate runs the failing tests`.

## Arbiter

The arbiter command refused, verbatim: `arbiter: stop not eligible: a Merge-In or Publish verify halt whose failure record names a target belongs to batten's retry and then to the parent; way forward: report the stop to the run's parent in the stop report, and \`lyx loom status <run-id>\` shows the stop`.

The driver did not run `goto`, `resume`, or any repair verb.
The `goto` in the way forward is an operator and orchestrator verb (see the driver prompt, `## Ways forward`).

## Earlier stops this attempt

- Batch 07 (Webster) is past: the parent's retarget and rebaseline were applied, and the loop advanced to Publish.
- Discussion-Bouncer rounds 3 and 4 (circling, then hard-cap) were both resumed by the arbiter earlier in the attempt.
