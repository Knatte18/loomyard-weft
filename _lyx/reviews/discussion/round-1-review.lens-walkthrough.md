```yaml
lens: walkthrough
outcome: reported
```

Walkthrough list (decision — case — purpose):

- D1 — `lyx board upsert` of project P with `depends_on` — refused; batten claims P only for its D11 landing — met.
- D2 — rename of member X + prune of a done member on run-held project P — `project`/`order` rewrites follow; run-lock exemption mirrors `depends_on` (`internal/boardengine/runlock.go:66`) — met.
- D3 — payload with `beam:true` on P — refused naming `lyx beam create` — met.
- D4 — unspawned member X of beam B, read on machine M2 — no origin for X → `ParentBeam` = B → `Effective(B)` = B's owner — met.
- D5 — member X of beam B while B's worktree has uncommitted tracked edits — start forks from B's last commit as stated; landing does not — **missed** (W1).
- D6 — `lyx fabric add main --adopt-local` from a prime on a beam branch — default branch → empty parent; `main-weft` checked out elsewhere refused naming its worktree — met.
- D7 — member X started between D9 steps 1 and 2 — step 1's owner write makes the precondition name `lyx adopt <beam>` — **missed** (W2).
- D8 — beam B's orch runs `lyx batten run X --window` — window in B's reed session, prime cwd, state under prime — met for this case.
- D9 — create of nested project P under beam B, stopped after step 1 — re-run sees flag set, `Add` with `Parent=B` — met on re-run, but the member way-forward in between is wrong (W2).
- D10 — promote of running task T, stopped after step 1 — T left an unclaimed task with its pair — **at risk** (W3).
- D11 — land of B into the prime — claim; Worktree-Create skips; Seed-Child seeds `beam`; describe writes; Finalize lands; Land prunes; Teardown removes the pair and marks P done — met; Finalize change stated.
- D12 — B lands — teardown's remove regenerates; marked `<repo>-B.code-workspace` goes — met.
- D13 — X in beam B with B's orch stopped — walk finds B not live → prime; notice reaches prime orch — met.
- D14 — upsert of new task `foo-weft` — refused with fabric's reason — met (owner-setter kind change in D10 is outside this lens).
- D15 — README for project P with an unordered member — renders after ordered by handle — met.

Findings:

**W1 — MEDIUM — design — D5 (and D7): beam members with uncommitted edits in the beam halt at landing.**
D5 accepts the beam's tracked dirt because "the operator works in the beam, so its dirt is routine", and its stated bound covers only member starts.
Every member lands via Finalize into the beam's own worktree, and Finalize is Stuck on "a pair that is missing, dirty or mid-merge" before any merge, so each member of a beam the operator is editing halts at Finalize — a routine stop the record does not state and which defeats D5's stated purpose.
The record should say whether Finalize keeps refusing a dirty beam parent (stating the stop as a bound) or what changes.
Evidence: `internal/landingshed/finalize.go:200` (Call doc); D5 in `_lyx/discussion/decision-record.md`.

**W2 — MEDIUM — decision — D7 with D9: the precondition's way forward is `lyx adopt <beam>` in exactly the case D9 creates, and following it builds a wrong beam pair.**
D9 step 1 records this machine as owner and sets `beam:true` before step 2 makes the pair; D3 makes the owner setter the only flag writer, so a verb-set beam always has an owner recorded.
D7 names `lyx beam create` only "when the board names no owner", so a member started mid-create (or after a create stopped between steps 1 and 2) is always told `lyx adopt <beam>`.
On the owning machine, adopt step 2 calls `Topology.Add` from the prime with no `Parent` and no beam mark; with no branch on origin, `Add` writes an origin with `ParentWorktree` = the acting worktree (prime), giving a beam pair with no beam mark (orch refuses in it, workspace files misgroup it) and the wrong parent when B nests in another beam.
The way forward should turn on whether the beam's pair/branch exists on origin, not on whether an owner is recorded, and name `lyx beam create <beam>` to finish a stopped create.
Evidence: `internal/adoptcli/doc.go` ("Step 2: the pair … Through `Topology.Add` from the prime"); `internal/fabricengine/add.go:440-441` (`WriteOrigin` with `ParentWorktree: l.WorktreeName` when no adopted weft carries a record).

**W3 — LOW — design — D10: a promote stopped after step 1 leaves a startable task.**
Step 1 ends the batten run and clears the claim and batten's records; until step 3 turns the entry into a project it is an ordinary open task with a live pair and the stopped run's weft, so an orch (or resume) can start it as a fresh loom run before the promote is re-run, whose step 1 would then cancel that new run.
The record states nothing about this window.
Evidence: D10 steps 1–3 in `_lyx/discussion/decision-record.md`; claim clear per `PATTERN-pair-teardown` ("a batten cancel's phase … leaves the board claim to the cancel, which clears it").

Tool calls:
- grep `internal/fabricengine/add.go` for origin-record writes on the adopt path — context lacked how `Add` records the parent when adopting an existing/remote branch, needed for W2's counterexample.
