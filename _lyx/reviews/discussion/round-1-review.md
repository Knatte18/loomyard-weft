---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D4; Auto-mode assumptions, dev-build entry)
    summary: D4 compares revision plus modified flag, but the dev-build assumption reasons from the revision alone, so the record states two incompatible rules for binary_changed
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D1 title; D2; Open risks, error-text entry)
    summary: D1's title says the transient mark is never text-matched, while D2's git-transport and github-api classes need Go-side inspection of git stderr and responses, which Open risks concedes
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Open risks, loom-start race entry)
    summary: The rule for a resume line arriving during an in-flight step job exists only in Open risks, and leaves open which report path and budgets the driver uses afterwards
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md (D5)
    summary: D5's git status guard names no repository or path scope and does not say that a skipped re-step waits for the next binary change
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md (D6 vs D10)
    summary: D6 starts the binary watch at every hand-back, while D10 runs it only while parked on an eligible stop
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md (D8, live driver without marker)
    summary: D8's behave-as-today branch is not spelled out, while its acceptance criterion reads neither sends nor spawns
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md (Constraints)
    summary: Constraints omits the Buildinfo Leaf Invariant, which bars the natural home (internal/buildinfo) for the new build-identity reader
  - id: F8
    severity: LOW
    location: _lyx/discussion/decision-record.md (Acceptance criteria, SKILL.md entry)
    summary: The SKILL.md acceptance criterion omits several skill-only behaviours from D6, D7, D9, D10 and D11, so they have no checkable criterion
  - id: F9
    severity: LOW
    location: _lyx/discussion/decision-record.md (Open risks, HANDOFF entry)
    summary: The HANDOFF entry is not a risk but a scope exclusion the plan writer needs, filed under Open risks
  - id: F10
    severity: NIT
    location: _lyx/discussion/decision-record.md (Notes for the plan writer)
    summary: The Notes spell a method parameter list, Runner.Send(guid, text), which exceeds the writer stencil's Step 2 bound on exact signatures
---

### [MEDIUM] D4 and the dev-build assumption state two different rules for `binary_changed`

**Location:** `decision-record.md`, D4 and the last Auto-mode assumptions entry.
**Issue:** D4 records "the VCS revision and modified flag" and sets `binary_changed` when "the running binary's identity and the recorded one are known and differ", so the modified flag is part of the compared identity.
The assumption then says "Dev builds from a modified tree share a revision with their base, so they trigger no binary-change re-step", which holds only if the flag is not compared.
Under D4 as written, a step recorded by a clean-tree build at revision R and a status read by a modified-tree build at R differ, and do re-step.
The plan writer cannot tell which rule to implement.
Evidence: the installed `lyx` carries both `vcs.revision` and `vcs.modified=false` (`go version -m`).
**Fix:** State in D4 that the identity is the pair (`vcs.revision`, `vcs.modified`) and that the pair is what is compared.
Rewrite the assumption to match: successive dev builds from a still-modified tree at one base revision compare equal and trigger no re-step; a build that moves the revision or flips the flag does.

### [MEDIUM] D1's "never text-matched" contradicts the Go-side classification D2 needs

**Location:** `decision-record.md`, D1 title, D2 classes 1 and 2, and the Open risks entry "Classifying git and GitHub failures by their error text inside lyx".
**Issue:** D1's title reads as a global ban on text matching.
Its rationale limits the ban to the skill ("The skill branches only on envelope fields … never on text"), and D2's `git-transport` class (connection refused, DNS failure, timeout) can only be told apart from a rejected push by reading git's stderr in Go, which the Open risks entry admits.
A plan writer taking the title at face value would refuse the one implementation D2 requires.
**Fix:** Retitle D1 so the ban names the driver, and state in its Decision where text inspection is allowed: at the Go failure site only, against the failed call's own output (git stderr, HTTP status), falling back to "not transient" on anything unrecognised.

### [MEDIUM] The in-flight resume rule is a decision filed under Open risks, and it is incomplete

**Location:** `decision-record.md`, Open risks, "A small race remains where `loom start` reads the marker just as the driver self-resumes".
**Issue:** "a resume line during an in-flight step job is noted and ignored" is a skill behaviour the plan writer must implement, but no Decision or Acceptance criterion states it.
It also leaves "noted" undefined: `loom start` has already minted a new report path and removed the marker, and D8 says the resume line resets budgets and names the next report path.
If the driver ignores the line entirely, the minted path is never written and the reset is lost; if it adopts them, the record should say so.
**Fix:** Move the rule into D8 as part of the resume handling: a resume line that arrives while a step job is in flight starts no second step, and the driver adopts its report path and budget reset for its next stop.
Keep a one-line residual in Open risks only if something remains uncovered.

### [LOW] D5's clean-tree guard has no stated scope, and its skip is sticky until the next deploy

**Location:** `decision-record.md`, D5.
**Issue:** The focus asked whether the driver's own hand-back writes can dirty the tree and permanently suppress the re-step.
They cannot, if the guard runs on the code worktree: `_lyx` there is a symlink into the weft sibling (`ly-drive-safe-repairs-weft/_lyx`), both `_lyx` and `.lyx` are excluded through `.git/info/exclude` (Fabric Git Invariant), and stop reports, status and records all live under them; `git status --porcelain` in this worktree is clean.
The record does not say which repository the guard reads, so a plan writer could run it in the weft, where the driver's stop report is dirty until `commit-records` lands, or where a failed push leaves it so.
Separately, D10 re-arms the watch after a failed guard, so a skipped re-step does not retry when the tree turns clean; it waits for the next binary change.
That is a reasonable outcome but is stated nowhere.
**Fix:** State in D5 that the guard reads the task's code worktree only (the warp), never the weft `_lyx` or `.lyx`, and that a skipped re-step is not retried until the next binary change or a resume.

### [LOW] D6 and D10 disagree on when the watch runs

**Location:** `decision-record.md`, D6 and D10.
**Issue:** D6 says the parked driver "starts its binary watch as a background job" at every hand-back; D10 says the watch runs "while parked on an eligible stop".
For `awaiting` or `paused` the two give different behaviour.
**Fix:** Make D6 start the watch only when the stop is D5-eligible.

### [LOW] D8's "behave as today" branch is left implicit

**Location:** `decision-record.md`, D8, second bullet.
**Issue:** Today a live driver strand makes `mustSpawnDriver` false (`internal/loomcli/bootstrap.go`), so `loom start` spawns nothing and goes straight to its handover.
The acceptance criterion reads that branch as "neither sends nor spawns", which matches, but D8 itself leaves the plan writer to reconstruct it.
**Fix:** Spell the branch out in D8: no `Send`, no spawn, then the usual handover (attach, or the envelope under `--no-attach`).

### [LOW] Constraints omits the Buildinfo Leaf Invariant

**Location:** `decision-record.md`, Constraints.
**Issue:** `internal/buildinfo` is the package a plan writer would reach for to read build identity, but the Buildinfo Leaf Invariant makes it import nothing and expose `Channel`/`IsDev()` only, so `runtime/debug.ReadBuildInfo` cannot live there.
The other named invariants all exist and say what the entries claim: Shed Producer-Seam (stdlib, `state`, `lock`), Shed Verb-Set (five kinds; key sets closed by doc comment and test), Shed Run-Directory and Lyxdirs Single-Declarer, Durable-vs-Ephemeral, Shuttle Provider-Seam, Driver Choice Single-Site (the resume branch reads strand liveness and the marker, not the seed's driver field), Live-Substrate Spawn Observability, CLI/Cobra and Documentation Lifecycle.
**Fix:** Add a Buildinfo Leaf Invariant entry: the build-identity read stays out of `internal/buildinfo`.

### [LOW] Skill-only behaviours lack a checkable acceptance criterion

**Location:** `decision-record.md`, Acceptance criteria, the `SKILL.md states …` entry.
**Issue:** For skill-only behaviour, "SKILL.md states X" is the intended check, and it is sufficient if X is listed.
The entry covers D3, D5, parking, the watch, resume handling and repair records in general terms, but omits: the driver removing the marker before a self re-step and treating the marker on disk as the truth on every wake (D7); the per-attempt report rewrite listing every automatic re-step (D9); watch re-arm on timeout or on a failed confirmation, and stopping the watch before a resume step (D10); the repair-record fields (D11); and orchestrator forks and operator sessions not parking (D6).
**Fix:** Enumerate those items in the `SKILL.md` criterion.

### [LOW] HANDOFF entry is a scope exclusion, not a risk

**Location:** `decision-record.md`, Open risks, last entry.
**Issue:** Completeness-before-leanness: the entry carries a constraint the plan writer needs — do not edit `manifest/HANDOFF.md` — which belongs in Scope Out.
As a risk it states nothing that can go wrong in this task's delivery.
**Fix:** Add `manifest/HANDOFF.md` to Scope Out as operator-owned and drop the Open risks entry; update the Q15 ledger note, which points at Open risks.

### [NIT] Notes spell an exact signature

**Location:** `decision-record.md`, Notes for the plan writer, second bullet.
**Issue:** The stencil's Step 2 bars gathering exact signatures.
File names and function names are coarse pointers and within bound; `Runner.Send(guid, text)` carries a parameter list.
**Fix:** Write it as `shuttleengine.Runner.Send`.

## Verdicts on focus entries that yielded no finding

- Focus 4, state machine: D3, D5, D8 and D10 are consistent once F3, F5 and F6 are fixed; the resume reset of "repair and re-step budgets" is coherent, since the binary-change bound is derived and has no counter to reset.
- Focus 7, relocation: D5's 2026-09-30 history is rationale, which the stencil places in Decisions, so it is not a relocation candidate.
  The Auto-mode assumptions that restate D2 and D5 record the autonomous picks with their reasons, which is that section's contract; not a defect.
- Focus 8, Notes: beyond F10, the named files and functions are coarse pointers within the Step 2 bound.
