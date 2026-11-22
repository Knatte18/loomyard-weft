# Drive the four teardown and archive cases the one-repository hardener left undriven

`harden-fabric-undriven-cases` · Note · bug, fabric

## Seen

`harden-fabric-single-repo-verbs` approved in round 3 with part of its focus undriven (`round-3-focus.md` and `round-3-review.md` in its archived records, Coverage section):

- The discard half of a killed teardown, re-run through the command its pending record prints; only a hubforge test with a Topology seam covers it.
- Whether a teardown now pushes to origin a local `archive/<slug>/*` tag the operator kept local on purpose, and whether any verb output or help text names it.
- A failed remote step without a kill keeping its pending record (O3's fix).
- `lym adopt --archives` over owner-archive tags pushed by the real `ArchiveOwnerTips` path, with the code side's new `refs/lym/code-tags/` prefix.

Also left: code-side archive tags in plain `refs/tags/archive/` that an earlier binary pushed are named by no reader verb.

## Idea

A short live drive of these cases in a one-repository sandbox hub, fixing what fails with a test.
