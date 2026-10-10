# Bounce budget blocks a review segment after its round was approved, before the Bouncer verdict

`bounce-budget-after-approved` · Note · bug

Imported from [issue #503](https://github.com/Knatte18/loomyard/issues/503).

Seen (publish-recovery, 2026-10-10): Webster-Review's round 3 review was APPROVED (one LOW finding) and its fixer finished with a green verify gate, but the run blocked with "bounce budget exhausted for Webster-Burler" before the Bouncer wrote `round-3-bouncer-verdict.md`.
`lyx loom circling accept` refused, since the run was blocked rather than awaiting, so the orchestrator needed an investigation fork, then `lyx shed goto publish-recovery --to Webster-Burler` and `lyx loom resume`, which spent another review round on approved work.

Workaround: goto Webster-Burler plus resume.

Expected: the bounce budget is checked before a round starts, not after an approved round ends, so a round whose review approved always reaches its Bouncer verdict and the run moves on.
