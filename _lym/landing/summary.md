# Report on three one-repository fabric cases no hardener round drove

Drives the three cases the second fabric hardener campaign left untried, on a one-repository hub cloned from a local bare origin, and lands the findings as `docs/research/single-repo-undriven-cases.md`.
No code changes.

## What the drive found

- **A discard or remove killed mid-operation.**
  The detached `lym fabric remote-teardown` child outlives a kill of the foreground `lym batten cancel <slug> --remove` and holds the teardown lock.
  After that child is killed too, `lym fabric status` lists the pending record, and re-running `lym batten cancel <slug> --remove` finishes the teardown and clears it.
- **A fabric step that fails with no kill.**
  A rejected origin branch delete leaves a `discard` record carrying `rerun: lym batten cancel <slug> --remove`, and the re-run recovers.
- **`lym adopt --archives`.**
  Over owner archive tags pushed by hand, `lym adopt --archives root` lists each tag whose commit the slug's branch on origin lacks, on the code and records sides, and skips a tag the branch contains.
- **One fault.**
  The killed run's `remove` record names `lym fabric remove <slug> --remote` as the way forward, which answers `pair not found` once the pair is gone.
  It is filed as the board note `remove-record-rerun-pair-gone` (B1).

## Design decisions

- The hub was cloned by the orch from a local bare origin under `.scratch`, per the parent's recorded decision, so no GitHub repository was touched.
  The orch also added the pairs, since `lym fabric clone` and `lym fabric add` are refused from a task session.
- The owner archive tags were pushed by hand, because a real takeover needs a second machine; the report lists this under "Not driven".
- A discard-kind record from a kill was not reached: `lym batten cancel` runs the pair's remove first, so the killed record was of kind `remove`.

## Merge danger

**Door:** two-way
**Blast radius:** docs
