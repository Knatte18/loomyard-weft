---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/fabricengine/anchormove.go (CurrentSHA error return after the commit), internal/fabricengine/anchorchange.go (moveWeft, the empty-sha rollback)
    origin: lens:code-correctness
    summary: moveWeftContent returns an empty sha when CurrentSHA fails after the move commit landed, so moveWeft rolls back a landed change and leaves the worktree wired at A over content at B with no record
    fix: In moveWeftContent, once the commit has succeeded, never return an empty sha with an error; return the commit's sha, or a distinct landed-but-unread result, so moveWeft goes on as landed (keeps the record and names the re-run) instead of calling rollback. Add a row to TestAnchorChange for a failing sha read after the commit.
  - id: F2
    severity: LOW
    class: design
    location: internal/fabricengine/moverecord.go (MoveInProgress, the lock-held branch)
    origin: lens:code-correctness
    summary: MoveInProgress names the record's pid as the holder whenever the move lock is held, where D7 says to name the run as starting when the record's pid and start time are not the live holder's
    fix: In the lock-held branch of MoveInProgress, apply the same holder check heldMoveRefusal in anchorchange.go applies (record pid and start time against the live holder), and word the stop as starting when they do not match. Share the check between the two instead of duplicating it. Add a row for a held lock over a stale record.
  - id: F3
    severity: LOW
    class: design
    location: internal/lyxcwd/anchor.go (readRecordedAnchor, the own-marker ReadFile)
    origin: lens:code-correctness
    summary: An own marker that exists but cannot be read (a directory, no read permission) is treated as absent, so the worktree silently resolves to the Bolt marker or "." which is the rewire-at-repo-root outcome D2 rules out for a truncated marker
    fix: In readRecordedAnchor, treat a ReadFile error other than not-exist on the own marker as an error naming the file, as an invalid marker is. Add a row to anchorread_test.go for an own marker that is a directory.
---

### [LOW] moveWeftContent rolls back a change whose commit already landed

**Location:** internal/fabricengine/anchormove.go, internal/fabricengine/anchorchange.go
**Issue:** After the weft commit succeeds, a failing `CurrentSHA()` returns `"", err`. `moveWeft` reads an empty sha as "not committed" and runs `rollback`, which moves `.lyx` back to `A`, rewires `A` and removes the move record. The weft's `HEAD` already holds the marker `B` and the content at `B`, so the worktree is wired at `A` against content at `B` with no record to recover from. Unlikely, since it needs a git read to fail right after a commit, but it breaks D7's recovery invariant.

### [LOW] MoveInProgress misnames the holder of a held lock

**Location:** internal/fabricengine/moverecord.go
**Issue:** In the lock-held branch, `MoveInProgress` reports the record's pid. When a new run holds the lock and has not yet rewritten the record, that pid is a killed run's or zero. The pull, push, origin-reconcile and dobby-gate stops built on it print a wrong pid. D7 says to name the run as starting in that case, and `heldMoveRefusal` does.

### [LOW] An unreadable own marker falls back silently

**Location:** internal/lyxcwd/anchor.go
**Issue:** `readRecordedAnchor` goes on to the Bolt marker or `"."` on any `ReadFile` error of the own marker. A marker that exists but is unreadable therefore resolves as if absent. D2 only names invalid and empty markers, so this is a gap, but it reopens the repo-root rewire D2 exists to prevent.

## Rejected

- Fork finding (code-pattern-fit, LOW consistency): `hubforge.ReanchorPair` writes and commits the marker with raw git outside `fabricengine`. Rejected: `hubforge` is a test-fixture package, the plan's Shared Decisions license this fixture explicitly, and the mechanical PATTERN scans pass on the tree.
- Fork finding (code-correctness, LOW design): the integration drops a mode-only local commit. Rejected: the fork read the code without a concrete case, and a mode-only change to tracked weft content is not a flow the record names; it does not clear the bar for a finding.
- Fork gaps (code-affected-set, code-tests): no findings; the unread areas they list (launcher move test, `docs/overview.md` context, test bodies) were not turned into findings on suspicion alone.

My own holistic read covered the `lyxcwd` own-marker read and `NamedWorktree`, the declared `PATTERN-cwd-resolution` change (landed verbatim in `PATTERN.md`, background file updated, entry under the line cap), `hubgeom.DobbyGeometry` and `DobbyStart.MoveInProgress` (a pair with an invalid marker errors naming the worktree; the move records of the pair and the prime both gate), and `hubforge.ReanchorPair`. I found no further defect there. I did not read the individual plan card files, and I did not read the diff of the refusal guards, the clone/reconcile/add code or the CLI verbs myself; those rest on the forks' reads.
