---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D11 (and D9, D10)
    summary: lyx beam land cannot start, since batten's start guard refuses a run with no status file over a present pair and names cancel --remove, which would destroy the beam
    fix: >-
      In D11, after "Batten's start guard applies the first three refusals to a project entry", add a sentence:
      the start guard's refusal of a pair left from an earlier run (a run with no status file whose task worktree, records worktree or records branch is present) gives way for a project entry whose `beam` is true and whose local pair carries the beam mark in its origin record, so the landing reaches Worktree-Create, which finds the pair present and skips;
      every other entry keeps that refusal.
      In D11's acceptance criterion, add: a `lyx beam land` of a beam made by `lyx beam create`, and of one made by `lyx beam promote`, passes batten's start guard and reaches Worktree-Create's skip.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D5, D7
    summary: Add refuses tracked dirt in the acting worktree, which D5 and D7 make the prime, so a dirty prime still refuses every beam member's create
    fix: >-
      In D5, after "It does not refuse tracked dirt in a named parent's worktree", add:
      with `Parent` named, Add checks no tracked dirt at all, neither the named parent's nor the acting worktree's, since the pair forks from the parent's last commit and never from the acting worktree;
      with `Parent` empty, Add keeps today's refusal of the acting worktree's tracked dirt.
      In D7's acceptance criterion, add: a member of a beam starts and gets its pair while the prime holds tracked dirt.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4; Constraints PATTERN-owner-machine
    summary: D4 never says how ParentOf turns a found origin record into a parent slug, and today's ParentSlug answers the prime when the parent beam has no live worktree here, so such a member reads as prime-owned
    fix: >-
      In D4, after "Origin found with an empty parent reads as the prime and never falls through to the board", add:
      a found record's parent slug is its `ParentWorktree`, read as the prime only when that names the prime's worktree, and never derived by matching `ParentBranch` against the worktrees present on this machine;
      so a member whose beam pair is absent here still reads the beam as its parent, and its owner as the beam's owner.
      In D4's acceptance criterion, add: a member pair whose beam pair is absent on this machine reads the beam as its `parent_beam` and the beam's owner as its own.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D8
    summary: The wider guard admits every session in a beam worktree, including the beam landing's own Finalize agents, and D8 names that run as the exception without bounding it
    fix: >-
      In D8, replace "The beam test bounds it: an agent of a run works in that run's own pair, so a member's agents stay refused, and the only run whose agents work in a beam is the beam's own landing (D11)." with:
      "The beam test bounds it: an agent of a run works in that run's own pair, so a member's agents stay refused;
      the only run whose agents work in a beam is the beam's own landing (D11), and the guard refuses in a beam whose board entry is run-held by its landing, naming the landing, so the landing's merge fixer and conflict session reach no batten, adopt, cleanup or beam verb."
      In D8's acceptance criterion, add: a batten verb run from a beam whose entry is run-held by its landing is refused naming the landing.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D1, D15
    summary: Board layering reads every non-note entry as a task, so a project with no open dependency lists as Ready, and starting it as one is a beam's landing
    fix: >-
      In D1, after "Only a task is claimable, bar the landing run of a project whose `beam` is true (D11).", add:
      the board's layering gives a project no task layer: an open project lists in `lyx board list` and `get` under a layer of its own, `Project`, never Ready or a dependency letter, and no project counts as a dependency's depth.
      In D1's acceptance criterion, add: an open project with no members lists under layer `Project`, never Ready.
  - id: F6
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D11
    summary: D11 drops Finalize's large-merge reroute for the beam without stating that a conflict-session resolution against the parent's newer work then lands unreviewed
    fix: >-
      In D11, after "The beam's Finalize declares no `on_reroute` and is built so that it never reroutes: ...", add a Bound sentence:
      what this lets through is the conflict session's resolution of the beam against its parent's newer work, which no member's review saw;
      it is bounded by the `finalize` gate over the merged tree and by the fixer's size and kind caps, past which Finalize halts for the orch.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D11 first refusal; D9
    summary: The open-member refusal uses ParentBeam, so members D9 says keep the prime as parent still block the beam's landing until they land into the prime
    fix: >-
      In D11's first refusal, replace "while any open entry's `ParentBeam` is the beam" with "while any open entry's `ParentOf` answer is the beam",
      so a member spawned under a grouping project before the beam existed, whose origin record names the prime, does not hold the landing.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Acceptance criteria D10
    summary: The D10 criterion says a promote stopped after any step from 2 on leaves an entry no batten start accepts, but after step 5 the entry is a beam project whose batten start is its landing
    fix: >-
      In the D10 acceptance criterion, replace "a promote stopped after step 2 or any later step leaves an entry no batten start accepts, naming `lyx beam promote`, and a re-run finishes it" with
      "a promote stopped after step 2, 3 or 4 leaves an entry no batten start accepts, naming `lyx beam promote`, and a re-run after any step finishes it".
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3 Bound
    summary: D3's bound says D9 admits the create over a hand-set beam flag, but with no owner recorded the project reads as the prime owner's, so on any other machine D9 refuses it naming adopt
    fix: >-
      In D3's Bound, replace "and D9 admits that create and makes the pair" with
      "and D9 admits that create on the machine the project's effective owner names, and elsewhere refuses it naming the owner and `lyx adopt <slug>`".
  - id: F10
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D13
    summary: The ready-notice walk starts from the worktree of a task's ParentBeam, and D13 does not say where it goes when that beam's worktree is absent on this machine
    fix: >-
      In D13, after "The Land row's ready notice names each newly ready task to the orch the same walk finds from the worktree of that task's `ParentBeam`, the prime's for an empty one.", add:
      a beam whose worktree is absent on this machine sends the notice to the prime's orch, since this machine holds no orch of that beam.
  - id: F11
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Scope, D8, Acceptance criteria D8 and D13
    summary: The beam orch's strand is written ly-colon-slug-colon-orch, hard-coding this hub's shortname, while agentname.Format builds shortname-colon-slug-colon-orch
    fix: >-
      Replace each `ly:<slug>:orch` and `ly:<beam>:orch` in Scope, D8 and the D8 and D13 acceptance criteria with `<shortname>:<slug>:orch` and `<shortname>:<beam>:orch`.
---

### [BLOCKING] The beam landing is refused by batten's start guard

**Location:** `_lyx/discussion/decision-record.md` D11, with D9 and D10.

**Issue:** `lyx beam land` starts `lyx batten run <slug>` on a project whose pair already exists and which has no batten status file: `lyx beam create` makes the pair outside batten, and promote's step 3 clears batten's records.
Batten's start guard refuses a `run` or `step` with no status file when any task worktree, records worktree or records branch is present (`internal/battencli/startguard.go:78-85`), naming `lyx batten cancel <slug> --remove`.
That refusal fires before any row, so the landing never reaches the Worktree-Create skip D11 relies on, and its way forward would remove the beam.
D11 adds refusals to the start guard for a project but grants no exemption from this one.
All four lenses found it.

### [MEDIUM] A dirty prime still refuses a beam member's create

**Location:** `_lyx/discussion/decision-record.md` D5, D7.

**Issue:** `add` refuses `ErrSourceDirty` on the acting location's tracked dirt as its first check (`internal/fabricengine/add.go:188-193`), and Worktree-Create surfaces it as a precondition.
D5 and D7 pass the prime as the acting location whenever the board picks the parent, so tracked dirt in the prime stops every beam member's create.
D5 exempts only the named parent's dirt and D7 says no dirt check applies to a pair forked from a beam; neither says what Add does with the acting worktree's dirt when `Parent` is named.
Found by existing-paths, bound-breaker, walkthrough and interactions.

### [MEDIUM] ParentOf's reading of a found origin record is unstated

**Location:** `_lyx/discussion/decision-record.md` D4; the PATTERN-owner-machine constraint.

**Issue:** D4 falls through to `ParentBeam` only when no record is found, but never says how a found record becomes a parent slug.
Today's `fabricengine.ParentSlug` matches `ParentBranch` against live worktrees and returns "" when none holds it (`internal/fabricengine/origin.go:139-157`).
A member adopted onto a machine where its beam has no pair then reads as a prime member: `Owners.Effective` answers the prime's owner (`internal/boardengine/owners.go:118-125`), against the constraint that an unrecorded owner reads as its parent's.
`ResolveParent` reads `ParentWorktree` instead (`internal/hubgeom/parent.go:83`), which survives the parent's absence.
Walkthrough raised the same gap at LOW; merged here.

### [MEDIUM] The beam guard admits the landing's own agents, unbounded

**Location:** `_lyx/discussion/decision-record.md` D8.

**Issue:** D8 lets every batten verb, `lyx adopt`, `lyx cleanup` and every `lyx beam` verb run from any session in a beam's worktree.
The beam's landing runs darn's Finalize in the beam pair, whose merge fixer and conflict session are agents working in that worktree (`internal/landingshed/doc.go:33`), and no shell-guard row or agent setting denies those verbs to agents (`internal/shellguard/rows.go` has no batten row).
D8 names this run as "the only run whose agents work in a beam" but states nothing that limits what those agents can then do across the hub, which the rubric's attack-surface question requires.

### [MEDIUM] Projects layer as Ready on the board

**Location:** `_lyx/discussion/decision-record.md` D1, D15.

**Issue:** `ComputeLayers` gives every open, non-isolated entry with no open dependency the Ready layer (`internal/boardengine/layer.go:22-27,138-143`), with no kind test, so a project lists as Ready in `lyx board list` and `get`.
By D1 and D11 a `lyx batten run` on a beam project is its landing, which merges the beam into its parent and tears it down.
The orch role stencil starts only tasks a landing notice names and holds every other Ready task for the operator (`contracts/stencils/orch/orch-template-role.md:138-142`), and the Land row's `ReadyTasks` filters tasks, so no automatic start follows; existing-paths filed it BLOCKING, lowered here to MEDIUM for that reason.
The record still leaves the project's layer unstated outside the README, and a project shown as Ready invites exactly the start that lands a beam.

### [LOW] The dropped large-merge reroute has no stated bound

**Location:** `_lyx/discussion/decision-record.md` D11.

**Issue:** Finalize's large-merge reroute sends a conflicted or overlapping merge to review (`internal/landingshed/doc.go:36`, `finalize.go:871-883`).
D11 builds the beam's Finalize to never reroute, justified by "each member was reviewed", but the conflict session's resolution of the beam against the parent's newer work is reviewed by no member.
D11 does not say that this is what lands unreviewed, or that the `finalize` gate and fixer caps are what bound it.

### [LOW] The open-member refusal counts members that land into the prime

**Location:** `_lyx/discussion/decision-record.md` D11 first refusal, D9.

**Issue:** D9 says members spawned under a grouping project before it became a beam keep the prime as parent and land there.
D11's first refusal reads `ParentBeam`, which names the beam for them, so they hold the beam's landing until they land elsewhere, while D4 makes `ParentOf` the answer for where an entry's pair lands.

### [LOW] D10's acceptance criterion contradicts D1 and D11

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, D10.

**Issue:** The criterion says a promote stopped after step 2 "or any later step" leaves an entry no batten start accepts.
After step 5 the entry is a project with `beam: true`, which D1 and D11 make startable as its landing, and D10's own start-guard refusal covers only a task.

### [LOW] D3's bound holds only on the prime owner's machine

**Location:** `_lyx/discussion/decision-record.md` D3 Bound.

**Issue:** A hand-set `beam: true` project with no recorded owner reads as owned by the prime owner (`internal/boardengine/owners.go:104-125`).
On any other machine D9 refuses the create naming adopt, so "D9 admits that create and makes the pair" overstates the bound.

### [LOW] The ready walk has no answer for a beam absent on this machine

**Location:** `_lyx/discussion/decision-record.md` D13.

**Issue:** The ready notice walks from the worktree of the task's `ParentBeam`; when that beam has no worktree here there is no origin record to walk from (`internal/fabricengine/origin.go:121-126`), and D13 names the prime only for an empty `ParentBeam`.

### [NIT] The beam orch's strand name hard-codes this hub's shortname

**Location:** `_lyx/discussion/decision-record.md` Scope, D8, acceptance criteria D8 and D13.

**Issue:** `agentname.Format` builds `<shortname>:<slug>:<role>` (`internal/agentname/agentname.go:102-115`); `ly` is this hub's shortname, so `ly:<slug>:orch` misnames the strand on every other hub.

## Rejected

- Walkthrough F3 (D4 parent field unstated, LOW): the same defect as bound-breaker's D4 finding; merged into F3.
- An older binary keeping an inherited `beam` key when it forks from a beam's weft (existing-paths, bound-breaker): covered by the Open risk that every machine deploys this build before the first project entry, so no new finding.
