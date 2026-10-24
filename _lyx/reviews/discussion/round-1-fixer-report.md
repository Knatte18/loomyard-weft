# Fixer report — discussion, round 1

Every finding's premise was checked against the code it cites and found to hold, so every finding is fixed.
Files changed: `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md` (the Round 1 entry, plus Q9, Q14 and one rejected-alternative line revised to the prime-and-beams guard).

## Changes

- F1 (fixed as proposed): D8's guard admits only the prime and a beam for batten verbs, `lyx adopt`, `lyx cleanup` and the `lyx beam` verbs, and refuses any other worktree with a named place to run the verb.
  D8 now says what the wider guard lets through and how the beam test bounds it.
  Scope In, D9 and the D8 acceptance line follow.
  The window placement was then revised in the self-check (see below).
- F2 (fixed as proposed): in D13, `ResolveParent` keeps the recorded parent worktree for parent-pair readers, Finalize's parent-records commit among them, and a separate hubgeom walk serves notice and parent-name readers.
  A D13 acceptance line covers the beam's run records.
- F3 (fixed as proposed): the PATTERN-pair-parent text is narrowed to choosing a new pair's parent and reading an owner through it; the entry stays within the 360-character `MaxEntryLineChars` cap.
- F4 (fixed as proposed, using the reviewer's refusal option): D11 refuses while the beam's orch strand or watcher is live, naming `lyx orch stop`, and refuses a land from inside the beam.
  D11 states where the land window opens, and has an acceptance line for it.
- F5 (fixed as proposed): D5's Bound keeps Finalize's dirty-parent Stuck, whose existing way forward is commit or stash in the parent pair (`internal/landingshed/finalize.go:406`); acceptance line added.
- F6 (fixed as proposed): D9 admits a `beam: true` project owned by this machine or by none, so a re-run finishes a stopped create.
  D7 picks its way forward by whether the beam's branch exists on origin.
  D3's Bound now names the way forward that results.
- F7 (fixed differently): the reviewer's first option, writing the board kind change before the cancel, would be refused by the run lock, since the entry is still run-held.
  I took the second option: promote's steps are reordered so the beam mark is written after the run's processes end and before its claim clears.
  Batten's start guard refuses a task whose local pair carries the mark, naming `lyx beam promote`.
  The new Bound says a promote stopped before step 2 leaves an interrupted-cancel state.
  The acceptance line is therefore scoped to a stop "after step 2 or any later step", not the reviewer's "after its first step", because before step 2 no persisted mark exists to refuse on.
- F8 (fixed as proposed): D4's `parent_beam` carries `ParentOf`'s answer; acceptance line updated.
- F9 (fixed as proposed, way forward revised in the self-check): D2's rename refuses a beam project and any entry with a local pair; acceptance line added.
- F10 (fixed as proposed): D14 drops the kind change from its checked writes, names `lyx board promote` of a note explicitly, and says `lyx beam promote` makes no slug.
  D3 says the owner setter also changes kind and clears the fields for promote.
- F11 (fixed as proposed): D1's dependency refusal judges only a write that sets or changes `depends_on`.
- F12 (fixed as proposed): refusals in D1, D2 (field refusals and the merge refusal), D9, D10, D11 and D14 now name their way forward in the Decision's own text.
  The acceptance criteria were rewritten as one line per Decision (D1 to D16), each reaching that Decision's refusals and rules, D12's gone-parent rule included.
- F13 (fixed as proposed, wording revised in the self-check): D13 routes a caller with no run to the prime's orch as today.
- F14 (fixed as proposed): D2 refuses a change of `project` on an entry with a local pair, naming `lyx batten cancel <slug> --remove`.
  D11's Bound names the case it still misses.
  I keyed the refusal on a local pair rather than "origin names a parent", because every pair's origin names a parent, the prime included (`internal/hubgeom/parent.go`).
- F15 (fixed as proposed): the cleanup skip wording moved out of D14 into a new D16, with a Scope In line.
- F16 (fixed as proposed): the Auto-mode assumptions are one-line pointers to D8 and D9, D11 and D2, with their ledger ids.

## Self-check

- fix-consistency MEDIUM, merged with fix-proposal MEDIUM 1 (D13's "acting from the prime" exemption caught batten's notices): fixed as proposed.
  The exemption now keys on acting on no run's pair, never on the location acted from.
- fix-consistency MEDIUM, merged with fix-proposal MEDIUM 2 (the orch refusal was checked only at `lyx beam land`, and an orch could start during the landing): fixed as proposed.
  Batten's start guard and Worktree-Create now apply the first three refusals.
  `lyx orch start` refuses in a beam whose entry is run-held by its landing; acceptance line updated.
- fix-consistency LOW (D11's "move its `project` off the beam" clashes with D2's new refusal): fixed as proposed; that option is now limited to an entry with no pair.
- fix-consistency LOW, merged with fix-proposal LOW 4 (D1's way forward named note members): fixed as proposed; it now names task members only.
- fix-claims MEDIUM 1 (`parent_beam` is right only on the machine holding the pair): fixed with my own change.
  The lens offered no change; I added a D4 Bound for other machines and scoped the acceptance line to the machine holding the pair.
- fix-claims MEDIUM 2 (a `--window` window for a non-member run could sit in a beam's session that is then torn down): fixed with my own change.
  `--window` now opens in the run's parent worktree's reed session wherever the verb runs, so a beam's session holds only its members' windows.
  D11's land uses that rule, and the Constraints line on `batten-window-default` follows.
- fix-claims LOW 3 (`lyx orch stop` leaves the watcher to exit on its own): fixed as proposed; the way forward adds a re-run once `lyx orch status` reports the watcher gone.
- fix-proposal LOW 3 (rename's `lyx beam land` way forward destroys the beam): fixed as proposed.
  The refusal now says a beam's slug stays fixed while it lives and names `lyx board upsert` of its title.
- fix-way-forward MEDIUM 1 (`lyx adopt` adds no pair for a never-started task): fixed as proposed.
  D10 refuses a never-started task with its own way forward: a new project entry, then `lyx beam create`.
- fix-way-forward MEDIUM 2 (no Decision lets adopt admit a project): fixed as proposed.
  D8 admits a beam project in `lyx adopt`, which takes over its owner and adds its pair when its branches are on origin; acceptance line added.
- fix-way-forward MEDIUM 3 (the guard's named parent can itself be refused or gone): fixed as proposed.
  The guard now names the nearest existing beam up the origin chain, else the prime.
- fix-way-forward LOW 4 (no takeover route for a beam whose owner never finishes the create): fixed as proposed; D9 also names `lyx adopt <slug>` and then `lyx beam create` here.
- fix-way-forward LOW 5 (unclear whether a note's slug can be reused): fixed as proposed; D9's way forward names a project entry under a slug of its own.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
