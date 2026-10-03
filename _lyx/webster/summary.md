# Pair lifecycle: teardown composite, remote branch sweep and retiring driver

This session resumed with every batch already `done`, so it ran only the integration stage.
The plan's `## verify:` command ran once at HEAD `d4d9f6b96` and the integration report says `status: OK`, with no deviations.

The completed batches cover these changes:

- `remove` gained a read-only refusal probe, commits pending sibling records before the archive, finishes half-removed pairs and returns a typed not-found.
- `remove --remote` deletes the landed task branch on origin.
- reed gained engine-less session end by name and a retiring mark.
- The new `pairteardown` composite chains quiet wait, refusal probe, session end and removal.
- batten's teardown closures and `lyx fabric remove` now go through the composite.
- The half-removed helpers were deleted.
- `cleanup --remote` sweeps leftover task branches on origin, composing open PR heads into its envelope.
- `loom start` replaces a retiring driver strand.
- The pair-teardown invariant, its tripwire and the docs landed.
