# Round 3 fixer report — discussion

## Changes

- F1 (BLOCKING), fixed as proposed, option (a).
  D10: the verify fixer runs through `gatefixer.SpawnOrAttach` and `gatefixer.Await`, with a Weaver-owned `gatefixer.Store` over the fixer records in Weaver's state.
  Its reply file `fix-<n>.md` is its only output file, and it carries no shuttle gates.
  D10 also states that `gatefixer` saves the record only after the spawn returns, one fixer per failed evaluation, that the fixer's own gate calls spawn none, and that a verify runner crash gets no fixer.
  D12: Weaver records the fixer number before the spawn, and a live `gatefixer` record is attached through `SpawnOrAttach`.
  A fixer number with no record is probed by reply path through `ProbeGated`; a strand found there is stopped and counted as a machinery loss, and the verify runs again.
  D9 adds the fixer numbers and records to state.
  The PATTERN-gate-slot constraint line, the D10 and D12 acceptance criteria and support-log Q9 and Q15 now agree with this.
- F2 (MEDIUM), fixed as proposed.
  D2: before the reset changes anything, it probes the strand of every batch attempt and fixer state records, whatever its outcome, by recorded output file through `ProbeGated`.
  `--stop-strand` stops every strand found through the handle's `Stop`.
  The reset acceptance criterion covers both cases.
  The self-check changed the wording for the outcome-recorded case; see Self-check.
- F3 (MEDIUM), fixed as proposed.
  D12: an `AttachGated` or `ProbeGated` error makes the row return Stuck naming the error, and nothing is spawned.
  D6: a failed stop returns Stuck naming the guid and `lym reed remove <guid>`, with the outcome already recorded so D12's first step stops the strand again; the arbiter meets a Stuck that names the live strand.
  `gatefixer.Await`'s stop-failure warning is treated the same way.
  The D6 and D12 criteria now name both returns.
- F4 (MEDIUM), fixed as proposed.
  D1 bounds the restore to a task-branch HEAD equal to the end commit the restored version records for its last batch, otherwise `lym weaver reset`, and gives the reason.
  To make that end commit checkable, D9 adds each ended batch's end commit to state.
  The D1 criterion now names the bound.
- F5 (MEDIUM), fixed as proposed.
  The two Open risks bullets are dropped.
  The D2 and D9 sentences are added.
  Q16 and Q17 are marked answered, and Q1 points to the answer.
  Open risks now holds a new question from the self-check (see Deferred) instead of `None.`.
- F6 (LOW), fixed as proposed: D6's failed-state-write sentence now attaches through D12 and judges a report on disk through the attach's result.
- F7 (LOW), fixed as proposed, with the destination refined by the self-check.
  D2 archives the run record in place, as Webster's `archiveStateFile` does.
  The files are renamed under UTC-stamped names inside `_lym/weaver/`, named through `lymdirs` and read by no reader, Describe's prior-generation readers included.
  The archive is committed with the plan restore through fabric.

## Self-check

- fix-consistency MEDIUM: the D12 criterion's opening still attached a guid-less fixer through `AttachGated`. Fixed as proposed: the opening now names batch attempts only, and the first step covers attempts and fixers.
- fix-consistency LOW: decision delivery to a fixer reattached through `gatefixer` was unstated. Fixed as proposed in D12.
- fix-consistency LOW: two support-log Turn 4 lines were stale. Fixed as proposed: revision notes added.
- fix-way-forward MEDIUM and fix-proposal MEDIUM (merged): D1's restore missed an attempt in flight that had not committed. Fixed with own change: D1 also requires that `lym reed status` lists no live Weaver batch or fixer strand. The proposed `ProbeGated` check by report path was not used, because the restored state does not name the in-flight attempt's report path.
- fix-way-forward MEDIUM: a reset over a corrupt state file missed the in-flight strand. Fixed: the reset also refuses while reed lists a live Weaver batch or fixer strand, naming `lym reed remove <guid>`.
- fix-way-forward MEDIUM and fix-claims MEDIUM (merged): the dirty-tree verify after a lost fixer, and D6's same-rung fixer retry spawning without a re-run. Fixed in D10: the verify runs again after every fixer, a lost one included, before the next fixer spawns. A verify that refuses an unclean tree counts as a failed evaluation whose findings are the uncommitted paths.
- fix-way-forward LOW: the way forward with no committed state version was underspecified. Fixed: it now also moves the weft's plan directory back, with git, to where it stood before the run's first commit.
- fix-proposal MEDIUM: the `ProbeGated` probe removes, rather than finds, an outcome-recorded strand, because the record is terminal and so respawn-eligible (`attach.go` `dispositionCandidate`, `removeSupersededStrands`). Fixed: D2 and its criterion say the probe removes that strand.
- fix-proposal MEDIUM: under option (a), the row's `gates` entries do not run over a fixer's commit. Deferred; see Deferred.
- fix-proposal LOW: archive directories nest across resets and are carried into the prior-generation slot. Fixed with own change: the files are renamed under stamped names, as Webster does, instead of moving into a nested directory, and no reader reads those names.
- fix-proposal LOW: D12's first step did not cover fixers. Fixed: it now stops attempts and fixers whose outcome is recorded.
- fix-claims LOW: `lym reed remove` was attributed to shuttle's stop verb. Fixed: it is now Weaver's own way forward.
- fix-claims LOW: the wording of what the probe removes was too narrow. Fixed together with the fix-proposal MEDIUM above.
- fix-claims LOW: a fixer number with no record has more causes than a crash between spawn and record. Fixed: D12 now names a crash before the spawn and a failed start too.
- fix-claims LOW: `gatefixer.Await` stops a strand only on a terminal result. Fixed: D6 says Weaver awaits until the result is terminal.

## Disputed

None.

## Deferred

- Self-check fix-proposal MEDIUM (raised by the F1 option (a) fix): the row's `gates` entries do not run over a verify fixer's commit, so the row can return Done with a row gate failing at HEAD.
  Each sound change has a cost the parent should weigh:
  - accept the bound;
  - spawn the fixer through shuttle with the row's gates, which needs a PATTERN changes entry for PATTERN-gate-slot (the review's option (b));
  - run a gated last-rung strand over the fixer's commit before Done, which adds a spawn.
  The question is recorded under Open risks and as Q18 in the support log.

## Rulings

None.
