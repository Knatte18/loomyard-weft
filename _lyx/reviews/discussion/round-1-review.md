---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D8 (Decision, batten, adopt and cleanup)
    summary: D8 widens the prime-only guard to any drivable worktree, task pairs included, and states no bound beyond the self-pair refusal, so an agent or operator in any task pair can run batten, adopt --all and cleanup --apply, and a --window run opened there dies when that pair is torn down
    fix: In D8, admit batten verbs, lyx adopt and lyx cleanup only in the prime and in a beam (a pair whose origin record carries the beam mark, the same test D8 already uses for lyx orch), refusing a task pair that is not a beam with a message naming the prime or the pair's beam as where to run it; state in D8 that this is what the wider guard lets through and that the beam test bounds it; then batten run --window's window lives in the prime's or a beam's reed session, which no landing of a member tears down while a member pair exists (D11's refusal). Add an acceptance line that a batten verb, adopt and cleanup are refused in a task pair that is not a beam.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D13
    summary: D13 redefines hubgeom.ResolveParent as a nearest-live-orch walk, but Finalize's parentRecordsCommitter (internal/loomcli/landingdeps.go:464) uses ResolveParent(l).Worktree to find the parent pair whose run records it commits before the parent-side merge, so a beam member with no live beam orch commits the prime's records instead of the beam's
    fix: In D13, keep the pair's recorded parent worktree (origin ParentWorktree) as the answer for every reader that needs the parent pair, Finalize's parent-records commit among them, and give the live-orch walk its own result (a separate field or function) that only notice and parent-name readers use; state in D13 which readers take which, and add an acceptance line that a beam member landing with the beam's orch stopped commits the beam's run records, not the prime's.
  - id: F3
    severity: BLOCKING
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md PATTERN changes section (PATTERN-pair-parent) against D5, D11, D13
    summary: The declared PATTERN-pair-parent text says a board entry's parent is read only through ParentBeam and hubgeom's one ParentOf, yet D11's land refusal and D13's ResolveParent walk read origin records directly, and D5 keeps merge-back, the parent lock and teardown reading the origin record
    fix: Narrow the PATTERN-pair-parent bullet's text to what the design routes through the two functions, for example "Picking a new pair's parent for a board entry or reading an entry's owner through its parent, only ParentBeam and hubgeom's one ParentOf, never the acting worktree", so origin-record reads of a live pair's parent (merge-back, parent lock, teardown, D11's refusal, D13's walk) stay allowed; keep the line within the PATTERN line cap.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D11 (with D8's beam orch)
    summary: D11 tears down the beam pair, which ends the beam's reed session and so its orch strand, while the orch watcher (a separate lyx orch watch process, internal/orchcli/spawn.go:30) is registered nowhere teardown reaches; the record never says what happens to a beam's orch at landing, and lyx beam land run from inside the beam is not covered by D8's self-pair refusal because its batten runs from the prime
    fix: Add to D11 an explicit step or refusal for the beam's orch, either `lyx beam land` refuses while the beam's orch strand or watcher is live, naming `lyx orch stop` in the beam's worktree, or land stops the beam's orch through orch's stop verb before starting the batten run; state in D11 how land places its window in the parent worktree's reed session given that D8 places `--window` in the acting worktree's session, and add an acceptance line that landing a beam with a live orch leaves no orch strand or watcher behind.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D5 (Bound)
    summary: D5 accepts a beam's tracked dirt because the operator works there, but every member lands through Finalize into the beam's worktree and Finalize is Stuck on a dirty parent pair (internal/landingshed/finalize.go:200), so each member landing halts while the operator edits the beam, a routine stop D5's bound does not state
    fix: In D5's Bound, state what a member's landing does with a dirty beam worktree; either keep Finalize's dirty-parent Stuck and say so as the bound (a member waits at Finalize until the operator commits or stashes in the beam, with the Stuck's way forward naming that), or decide a change to Finalize's parent sync for a beam parent; add the matching acceptance line.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D7 (precondition way forward) with D9 and D3
    summary: D7 names lyx beam create only when the board names no owner, but D9 step 1 records an owner before the pair exists, so a member started mid-create is told lyx adopt, whose Add from the prime writes a beam pair without the beam mark and with the prime as parent; after a hand-set beam flag (D3) it is told lyx beam create, which D9 refuses because it admits only a project whose beam is false, and the same admission rule contradicts D9's own claim that a re-run finishes a stopped create
    fix: In D9, admit a project whose beam is true and whose pair is absent on this machine and owned here, so a re-run finishes a create stopped after step 1; in D7, pick the way forward by whether the beam's branch exists on origin, not by whether an owner is recorded, naming `lyx beam create <beam>` when it does not and `lyx adopt <beam>` when it does; restate D3's hand-edit bound with the way forward that results.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D10 (steps 1 to 3)
    summary: D10 step 1 clears the claim and batten's records, leaving an open unclaimed task with its pair until step 3 makes it a project, so a stopped promote lets an orch or operator start a fresh loom run in the pair, whose re-run then resets the weft under that run and kind-changes a run-held entry the run lock refuses
    fix: In D10, hold the entry against a start between steps 1 and 3, for example by doing step 3's board write (kind change to project, beam true, owner) before step 1's cancel, or by refusing a batten start of an entry whose pair's origin record already carries the beam mark (written in step 2, moved before step 1); state the chosen order and the bound, and add an acceptance line that a promote stopped after its first step leaves an entry no batten start accepts.
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4 (parent_beam) with D8's stencil filter and D9
    summary: D9 keeps members spawned under a grouping project on the prime as their parent, but parent_beam is ParentBeam over the board, so once the project becomes a beam those members read parent_beam as the beam, and D8's stencil filter hides runs that land into the prime from the prime's orch and shows them to the beam's orch
    fix: In D4, carry as parent_beam the one ParentOf's answer (origin record first, ParentBeam only for an entry with no record), so a spawned entry's parent_beam is the parent its pair lands into; update the acceptance line on parent_beam to cover a member spawned before its project became a beam.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D2 (Rename)
    summary: D2 lets rename rewrite project and order references and refuses nothing for a beam, but a beam's branch is derived from its slug, so renaming a live beam project leaves ParentBeam naming a slug with no pair (every member stops at D7) and D12 reading the beam pair as gone
    fix: In D2, refuse a rename of a project whose beam is true, and of an entry whose pair exists, naming the reason (its pair's branch is derived from the slug) and `lyx beam land` or `lyx batten cancel --remove` as the way forward; add the refusal to the D2 acceptance line.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D14 against D10 step 3 and D3
    summary: D14 lists a kind change to project as a slug-making write the rule checks and says the owner setter makes no slug, yet D10 step 3 makes that kind change through the owner setter, and D3 scopes the owner setter's extension to the beam flag only
    fix: In D14, drop a kind change to project from the checked writes and state that promote's kind change keeps a slug that already names a pair, so the owner setter checks none; in D3, state that the owner setter also changes kind and clears depends_on, recipe, profile and run_settings for D10.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D1 against D10
    summary: D1 forbids any task depending on a project while D10 refuses a promote only for an open dependent, leaving a done task's depends_on naming a project with no stated rule for later writes to that task
    fix: In D1, state that the refusal judges only a write that sets or changes depends_on, as D2 does for project and order, so a done task's unchanged depends_on naming a promoted project refuses nothing.
  - id: F12
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Acceptance criteria section against D1, D2, D9, D10, D11, D12, D14
    summary: Acceptance lines were not derived from each Decision's refusals and rules, so D10's no-pair, owned-elsewhere and past-merge refusals, D9's refusals, D14's project-slug and rename write sites and D12's gone-parent rule are unreached, and several refusals (D1, D2, D9, D10 past-merge, D11 land) name no way forward although the PATTERN-refusal-way-forward constraint requires one
    fix: For each Decision, name each refusal's way forward in the Decision's own text, and extend that Decision's acceptance line so it reaches every refusal and rule the Decision states.
  - id: F13
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D13 (NotifyPrime callers)
    summary: D13 replaces every NotifyPrime with one notify of a run's resolved parent, but callers with no run, hibernate (internal/hibernatecli/deps.go:112) and fabric's push notifier (internal/orchcli/pushnotifier.go:46), have no run parent to resolve
    fix: In D13, state that a caller with no run, or acting from the prime, notifies the prime's orch as today, and that only a caller acting on a run's pair resolves its parent.
  - id: F14
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D11 (Bound) with D2 and D4
    summary: An entry cancelled without --remove is not run-held, so D2 lets its project change while its pair's origin still names the old beam; ParentOf then reads the old beam and ParentBeam the new one, and D11's land refusal misses that pair when it sits on another machine
    fix: In D2, refuse a change of project on an entry whose pair's origin record names a parent, naming `lyx batten cancel <slug> --remove` first, and extend D11's Bound to name the case it still does not see.
  - id: F15
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D14 last sentence against the Scope section
    summary: Scope Out puts the blocked-run cleanup half of issue 724 out of scope, and Scope In names only the slug rule, yet D14 changes lyx batten cleanup's skip message, a cleanup change unrelated to the slug rule
    fix: Add to Scope In a line for lyx batten cleanup's skip naming lyx batten cancel <slug>, and move D14's last sentence into its own short Decision or into that Scope line, so D14 holds the slug rule alone.
  - id: F16
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Auto-mode assumptions section
    summary: All three assumptions restate Decisions (orch through lyx orch with none started, D8 and D9; landing needs no writer or review, D11; notes as members, D2), though the support log says the section was reduced to picks with no Decision of their own
    fix: Replace each bullet with a one-line pointer naming the auto-picked Decision (D8 and D9, D11, D2) without restating its content, or drop the bullets the question ledger already records and leave the section with the remaining picks.
---

### [BLOCKING] D8's wider guard has no bound beyond the self-pair refusal

**Location:** `_lyx/discussion/decision-record.md` D8.
**Issue:** `RequireDrivableWorktree` is `RequireWarpWorktree` (`internal/fabricengine/fabric.go:162`) and admits every task pair.
No shell-guard row and no agent settings deny cover `lyx batten`, `lyx adopt` or `lyx cleanup` (`internal/shellguard`, `internal/shuttleengine/claudeengine/settings.go` deny only `lyx webster` in forks), so `RefuseUnlessPrime` was the only barrier.
Under D8 a loom agent in a task pair can run `lyx adopt --all`, `lyx cleanup --apply` or `lyx batten cancel` on another run.
`lyx batten run B --window` from task pair A opens B's driver window in A's reed session, which pair teardown ends when A lands (`pattern/PATTERN-pair-teardown.md`).
The rubric's attack-surface rule requires the Decision to state what the downgraded guard lets through and what bounds it.

### [BLOCKING] D13's ResolveParent change misdirects Finalize's parent-records commit

**Location:** `_lyx/discussion/decision-record.md` D13.
**Issue:** `parentRecordsCommitter` (`internal/loomcli/landingdeps.go:459-480`) resolves the parent pair as `hubgeom.ResolveParent(l).Worktree` and commits its run records before the parent-side merge.
Today that is the origin record's `ParentWorktree` (`internal/hubgeom/parent.go:35-53`).
D13 makes `ResolveParent` return the nearest live orch, the prime when none, so a beam member with a stopped beam orch commits the prime's run records and leaves the beam's, the merge target, uncommitted.
The record names this caller nowhere.

### [BLOCKING] The declared PATTERN-pair-parent text forbids reads the design makes

**Location:** `_lyx/discussion/decision-record.md` `## PATTERN changes`, D5, D11, D13.
**Issue:** The declared entry reads "Creating a pair for a board entry or reading its parent: only `ParentBeam` and hubgeom's one `ParentOf`".
D11's land refusal reads "any local pair's origin record names the beam as its parent", D13 walks origin records upward, and D5 keeps merge-back, the parent lock and teardown on the origin record; none goes through `ParentOf`.
Judged by its declared text, the design violates its own new entry.

### [MEDIUM] A beam's orch is left undefined at landing

**Location:** `_lyx/discussion/decision-record.md` D11, D8.
**Issue:** D8 runs a beam's orch in the beam pair's reed session with its own watcher; D11's Worktree-Teardown ends that session before removing the worktree, and the watcher, spawned by `internal/orchcli/spawn.go:30`, is not a run-registered process teardown ends.
Focus answer: `lyx beam land` run from inside the beam is not caught by D8's self-pair refusal, since its batten runs from the prime; the bookend holds for batten, but the beam's own orch session and watcher meet the teardown with nothing stated.
D11 also says land opens its window in the parent worktree's reed session while D8 puts `--window` in the acting worktree's, with no mechanism named.

### [MEDIUM] A dirty beam halts every member landing

**Location:** `_lyx/discussion/decision-record.md` D5.
**Issue:** Finalize is Stuck on "a pair that is missing, dirty or mid-merge" before any merge (`internal/landingshed/finalize.go:200`), and a member's parent pair is the beam the operator edits.
D5's bound covers only member starts.

### [MEDIUM] D7's way forward dead-ends around D9's step order

**Location:** `_lyx/discussion/decision-record.md` D7, D9, D3.
**Issue:** D9 step 1 writes the owner before step 2 makes the pair, so D7's "owner recorded → `lyx adopt <beam>`" fires in exactly the mid-create case.
Adopt's step 2 calls `Topology.Add` from the prime (`internal/adoptcli/doc.go`), which records the prime as parent and sets no beam mark.
After a hand-set flag (D3), D7 names `lyx beam create`, which D9 refuses for a project whose `beam` is already true; the same rule blocks D9's re-run after step 1.

### [MEDIUM] Promote leaves a startable task between its steps

**Location:** `_lyx/discussion/decision-record.md` D10.
**Issue:** After step 1 clears the claim, the entry is an open task with a present pair until step 3, so a start in that window runs a new loom run in the pair, and the re-run promote then resets its weft and is refused by the run lock at step 3.

### [MEDIUM] parent_beam misreads members spawned before their project became a beam

**Location:** `_lyx/discussion/decision-record.md` D4, D8, D9.
**Issue:** D9 says such members keep the prime as their parent, but `parent_beam` comes from `ParentBeam` over the board, so the stencil filter of D8 routes them to the beam's orch while they land into the prime.

### [MEDIUM] Renaming a live beam project strands its members

**Location:** `_lyx/discussion/decision-record.md` D2.
**Issue:** The board entry states a beam's branch is derived from its slug and `branch_prefix`; a beam project holds no run lock, so D2's rename passes, and `ParentBeam` then names a slug with no pair.

### [LOW] D14 and D10 disagree on the owner setter's kind change

**Location:** `_lyx/discussion/decision-record.md` D14, D10, D3.
**Issue:** D14 checks "a kind change to project" but exempts the owner setter as making no slug, while D10 step 3 makes that change through the owner setter, and D3 extends the owner setter only with the flag.

### [LOW] D1's dependency ban against D10's open-only refusal

**Location:** `_lyx/discussion/decision-record.md` D1, D10.
**Issue:** A done task's `depends_on` can name a promoted project, and D1 states no unchanged-field exemption.

### [LOW] Acceptance criteria and refusal ways forward are not derived per Decision

**Location:** `_lyx/discussion/decision-record.md` `## Acceptance criteria`.
**Issue:** One method gap: D10 reaches only its dependent refusal, D14 only "a new task slug" at upsert, promote and rename, D12 not its gone-parent rule, and refusals in D1, D2, D9, D10 and D11 name no way forward.

### [LOW] D13 leaves run-less NotifyPrime callers unrouted

**Location:** `_lyx/discussion/decision-record.md` D13.
**Issue:** `internal/hibernatecli/deps.go:112` and `internal/orchcli/pushnotifier.go:46` notify with no run whose parent could be resolved.

### [LOW] A project change on a spawned entry splits the two parent readers

**Location:** `_lyx/discussion/decision-record.md` D2, D4, D11.
**Issue:** An entry cancelled without `--remove` takes a `project` change while its origin keeps the old beam, so `ParentOf` and `ParentBeam` disagree and D11's refusal misses that pair on another machine.

### [LOW] D14's cleanup sentence sits outside Scope

**Location:** `_lyx/discussion/decision-record.md` D14, `## Scope`.
**Issue:** Scope Out excludes the cleanup half of issue #724 while D14's last sentence changes `lyx batten cleanup`'s skip wording, and Scope In does not name it.

### [NIT] Auto-mode assumptions restate Decisions

**Location:** `_lyx/discussion/decision-record.md` `## Auto-mode assumptions`.
**Issue:** Each bullet repeats D8 and D9, D11 or D2; the support log's Turn 5 claims the section holds only picks with no Decision of their own.

## Focus verdicts

- D11's never-reroute rule: not a defect.
  The reroute sites in `internal/landingshed/finalize.go` are the large merge (878-884) and a kept fixer change with a review following (753); D11 names both outcomes, the gate for a large merge and a halt for a spent or over-cap fixer, and a nil `RerouteReviewed` already halts (`internal/landingshed/deps.go:305`).
  The rule is checkable through its acceptance line, and the fact that a reroute with no `on_reroute` fails the run is cited from `internal/shedengine/doc.go:96`.
- Board-entry rule: not a defect.
  D9, D10 and D11 write other entries, owners, kinds and statuses; no Decision changes `umbrella-groundwork`'s own body or scope.
- Relocation pass: apart from F15 and F16, the narrative in Decisions and Scope Out carries bounds the plan writer needs; Open risk 4 adds the upgrade effect on an existing hub, which D1 does not state, so it stays.

## Rejected

- bound-breaker F4 (reconcile fast-forwards a beam branch under a member's Finalize): unconfirmed by the lens itself, and it needs origin ahead of a beam its owner machine drives, which this design does not produce.
- bound-breaker F7 (D12's "every machine regenerates the same files" is false): the rationale means the same grouping rule without reading the board, and per-machine folder lists are the existing behaviour of the hub file.
- bound-breaker claim 22 (D1's `T` refusal unverified): `internal/boardengine/config.go:128` refuses a type label sharing the task handle letter, so the claim holds.
- interactions 8 (D8 `--window` against D11's window placement): folded into F4.
