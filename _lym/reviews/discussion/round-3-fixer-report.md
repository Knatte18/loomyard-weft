# Fixer report — discussion round 3

## Changes

- F1 (BLOCKING), fixed as proposed.
  - D6 gains a records guard. Every write of `scope.json` or `judged.json`, by a drift row, `lym drift amend` or `lym drift defer`, is committed in one weft commit carrying a `Drift-Records` trailer.
  - The newest such commit holds the copies. Every drift row, the gate entry, `lym drift check` and the verdict read from those copies.
  - Drift-Verify, Drift-Recheck, the gate entry and `lym drift check` first check the current files against the trailer. At a row a mismatch is a Stuck naming `lym drift amend --restore`; the gate entry and `lym drift check` report it without failing. A run in flight with no such commit reads its files as they are.
  - D6's seam list gains "newest". Its rationale names the threat, and its Bound now states the true bound: a hand-made weft commit with the trailer, which the checks cannot tell from one lym made.
  - Matching changes:
    - D4: check results are committed per D6, and Done at once applies after the records check.
    - D7 and D8: the records check is added.
    - D10: `judged.json` is committed and read per D6.
    - D11: `--restore` covers the records. An amendment is committed in a `Drift-Records` commit. A Bound sentence covers a hand-written amendment.
    - D13: the verdict reads the copies.
    - Auto-mode assumptions: a D6 note on the records guard.
    - Acceptance bullets for D6, D10 and D11.
- F2 (MEDIUM), **fixed differently** in one part.
  - D9's snapshot now covers HEAD, the tracked tree, and untracked paths outside ignore rules (with copies) in both checkouts. It also covers a list of paths and content hashes for the current capture's scratch directory.
  - The snapshot lives under the hub's board-level `.lym`, beside the run process record, under a name `internal/lymdirs` declares. The worktree's `.lym` is a link into the weft side's `.lym`, as `ls -la` in the worktree shows.
  - The way forward now covers a modified or deleted untracked path (restored from the snapshot's copy) and a changed scratch directory (deleted, then resumed).
  - A rule now covers a lost snapshot.
  - The acceptance bullet and the Constraints line for `PATTERN-durable-vs-ephemeral-state` were updated.
  - Not made: the proposed content-hash list of the baseline directory in the snapshot. D7's baseline digest already detects a baseline change, and F3 now recomputes it after the judge returns. The way forward for that case is `lym drift defer`, which needs no path list. A second mechanism for the same check would state the guard twice. D9's Bound says the digest covers the baseline.
- F3 (MEDIUM), fixed as proposed.
  - After the judge returns, and before it closes the judge's mark, Drift-Verify recomputes the baseline digest. A mismatch is a Stuck naming `lym drift defer`.
  - The D9 acceptance bullet no longer lists the baseline among the changes that name paths and the restore. It adds the `defer` stop.
- Support log: a Round 3 entry under Review rounds.

## Self-check

Each line gives the lens, the severity, the finding and its disposition.

- fix-consistency, MEDIUM: the `command`-mode sentences in D2 and D8 lacked the records check. Fixed as proposed: both now run and report the records check.
- fix-consistency, LOW: D13 had no source for a run without a `Drift-Records` commit. Fixed as proposed: it falls back to the files as they are.
- fix-consistency, LOW, merged with fix-proposal 2: Open risks lacked the records in-flight gap. Fixed as proposed: a line was added.
- fix-claims, LOW: D9's Bound overstated what the digest covers. Fixed as proposed: it now says "on a run that has one (D7)".
- fix-way-forward, MEDIUM: a deleted scratch directory could never match the snapshot. Fixed as proposed: a scratch directory that is gone is left out of a resumed row's compare.
- fix-way-forward, MEDIUM: a non-restore `amend` or a `defer` could commit forged records. Fixed as proposed: every records write, by a row or a verb, starts from the newest commit's copies.
- fix-way-forward, MEDIUM: `--restore` had no seal copy to write on a run without a seal. Fixed as proposed: `--restore` writes back whatever failed its check.
- fix-way-forward, LOW: the judge Stuck did not say where the snapshot copies are. Fixed as proposed: the Stuck names the snapshot's directory.
- fix-proposal, MEDIUM: under the lost-snapshot rule, a judge could delete the snapshot and erase its own edits. Fixed as proposed:
  - A snapshot that is gone while the judge's mark is open is now recorded in `scope.json` as a Stuck. Its way forward is the parent or operator checking both checkouts, then `lym loom resume`.
  - That resume takes a new snapshot. Nothing drops the Stuck silently.
- fix-proposal, MEDIUM: an in-flight run with no `Drift-Records` commit commits a hand edit as genuine, and the Bound did not say so. Fixed as proposed: D6's in-flight sentence and its Bound state this, and Open risks lists it.
- fix-proposal, LOW: the hash was undefined for an absent `judged.json`. Fixed as proposed: the hash covers each file's presence as well as its content.
- fix-proposal, LOW: a crash inside lym before the records commit now needs the parent. Fixed as proposed: D6 calls it a Stuck only the parent or operator clears.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
