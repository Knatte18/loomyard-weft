# Going back from a beam's PR-Review to add work should keep its PR open and reuse it

`beam-pr-detour-keeps-pr` · Note · feature, pawl, landing, orch

Imported from [issue #57](https://github.com/Knatte18/loomyard/issues/57).

Seen in hub Models-LYMHUB, beam duplication-and-import-followups, 2026-11-04.

The beam's landing had opened PR #267 to main and waited at PR-Review. The operator then wanted one more task (a live viewer, PR #268) in the beam before landing. During that detour PR #267 was closed on GitHub (11:15 UTC), although nothing required it: an open PR from the beam branch to main picks up new commits on its own.

When the task had landed and the beam was flagged ready again, the way back was rough:
- The pawl run had failed at Beam-Await with 'step Beam-Await interrupted (loop-exited)', so the ready flag and lym beam land were not read until the orch ran lym loom resume by hand.
- The run then blocked on 'pull request #267 was closed on GitHub without being merged; way forward: reopen it on GitHub and run lym loom resume'.
- After the reopen, the landing overwrote the PR's hand-written title and body and summary.md with a flat title list (#56).
- A finished motif member left behind a pair record that blocks moving it out of the beam (#52).

Expected: a supported way to step a beam back from PR-Review to take in more work, which keeps the existing PR open and reuses it, waits in Beam-Await for the new members without failing, and on the next ready goes straight back to PR-Review on the same PR with its body kept. The orch role and stencils should say not to close the PR for such a detour.