---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:154
    summary: "D7 applies the owner check at every Finalize entry, so a Finalize re-entered after its parent-side merge halts on owned_elsewhere, contradicting the declared PATTERN-owner-machine exemption and D6's carve-out"
    fix: "In D7's first bullet, state that the owner check is skipped for a Finalize this machine's own landing has reached past the parent-side merge, decided by the test the origin reconcile already uses to drop such a run from the ended set (`finalizeEntered` in `internal/fabricengine`, which reads the pair's loom status, the local parent branch and the prime's batten status and counts an unreadable state as reached). In D6's carve-out paragraph name that same test as the detector of a run whose Finalize already merged into the local parent, so InnerRun and the landing row cannot disagree. Add one acceptance criterion: a Finalize re-entered after its merge, for example the re-step after a transient parent-push failure, with its entry owned elsewhere, does not halt at the owner check and finishes the push."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:109
    summary: "D5 step 1 and its adopt-back criterion rest on LivePair reading an owner-paused pair live, but LivePair reads any pair a live pause entry covers, slug scope included, as not live"
    fix: "In D5 step 1, replace the premise that LivePair reads a slug-scope owner pause's pair live with the actual behaviour and a decision. `pairteardown.HubSeamOptions` fills `HubPaused` from `record.Covering(slug)`, which matches a slug-scope entry too (internal/pauserecord/record.go:129-148), and `OriginSeams.LivePair` returns not live when `HubPaused` is true (internal/pairteardown/originseams.go:35-44, 73-82; doc.go:67). Pick one: (a) adopt builds its own seams whose `HubPaused` covers hub entries only, so a pair under an owner pause still reads live and the live-pair stop and `--drop-local` refusal hold; or (b) drop the live-pair stop for an owner-paused pair, since the reconcile then judges its branches and the branch-state stop already covers its local commits, and keep the live-pair stop for a pair with no pause. State which, and rewrite the second sentence of the adopt-back acceptance criterion at line 234 to match, so a test reaches it."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:141
    summary: "D6's repair for an adopter-first landing resets only the code branch and the halt note names it unconditionally, though the parent-side merge concludes a commit on both warp and weft and a refused parent push has other causes"
    fix: "In D6 (lines 140-142) and the matching Open risks bullet and acceptance criterion, state what happens to the parent pair's records branch: the parent-side merge concludes a commit on the warp side and then on the weft side (internal/fabricengine/doc.go, the lifecycle quartet paragraph), while Finalize's push publishes the warp branch only, so `git reset --keep origin/<parent-branch>` in the parent's worktree leaves a local merge commit on the weft side. Name the weft-side step, or record in Open risks that the next origin reconcile judges that records branch and stops with its own way forward, and say which. Also condition the halt note's reset advice on origin's parent holding the task's work, the test Finalize's already-landed path uses; otherwise the note keeps the existing by-hand reconcile reason, since a reset to origin's tip on any other refusal drops a landing nobody else made."
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:252
    summary: The declared PATTERN-owner-machine text says every start, resume, landing and wake is judged against origin inside ReconcileOrigin, but D3 and D5 judge a start from the pulled Bolt, an offline resume from the local Bolt and each post-push owner from a re-read
    fix: "Make the PATTERN text and the design agree, and cite the declared entry where the design departs. Preferred: amend the entry at line 252 of decision-record.md to say the owner verdict that gates a resume, wake or landing is judged inside `ReconcileOrigin`, a start judges the Bolt it just pulled, and a resume with origin unreachable, which D3 bounds, reads the local Bolt. In Constraints at line 193, name `PATTERN-owner-machine` beside `PATTERN-origin-reconcile` for those same reads, since the Constraint cites only the latter."
---

### [BLOCKING] A re-entered Finalize halts on the owner check despite the declared exemption

**Location:** `_lyx/discussion/decision-record.md:154`, with D6 lines 134-137 and the PATTERN changes entry at line 252.
**Issue:** The declared `PATTERN-owner-machine` text exempts the rest of a Finalize already past its parent-side merge, and D6 builds a carve-out on it.
D7 states the check without that exemption: Finalize checks, by the landing seam's verdict, before the parent-side merge.
The landing row runs `reconcileVerdict` at step 1b3 on every `Call` (internal/landingshed/finalize.go), and `pushParent` documents that a transient push failure returns an error so the driver re-steps, with the re-step's merge reading already up to date and going straight to the push.
So a re-step of a Finalize whose merge is already in the local parent re-runs the owner check, reads owned_elsewhere and halts, leaving the unpushed merge in the parent, the hazard the carve-out exists to avoid.
The reconcile already has a detector for this shape, `finalizeEntered`, which drops such a run from the ended set (internal/fabricengine/replay.go, doc.go).
Neither D4's precedence nor D7 applies it to owned_elsewhere, and D6 names no detector for the run it keeps the lock of.
Because the declared text binds, this is a violation of the listed constraint.

### [MEDIUM] D5's live-pair stop rests on a LivePair behaviour the code contradicts

**Location:** `_lyx/discussion/decision-record.md:109` and the adopt-back criterion at line 234.
**Issue:** D5 says a slug-scope owner pause leaves the old driver live and that `LivePair` reads such a pair live, so adopt stops the slug before the owner write.
`HubSeamOptions.HubPaused` calls `record.Covering(slug)`, and `covers` matches an entry of the slug's own scope as well as a hub entry.
`OriginSeams.LivePair` returns not live when that is true, on the stated ground that a paused pair's driver is interrupted.
The `pairteardown` doc and the `HubPaused` comment say "hub pause entry", which is how round 2's premise arose, but the code behind the name covers the slug scope as well.
Under the stock seam, the pair the criterion describes ("still live under its owner pause, the pair reads live") reads not live.
The reconcile then judges its branches and the branch-state stop still catches local commits, but the live-pair stop and the `--drop-local` refusal are unreachable for the owner-paused pair, and `--drop-local` would move a branch under strands that are alive though interrupted.
The Decision does not say which behaviour adopt wants or how its seam gets it, so a plan following the text builds a criterion no test can reach.

### [MEDIUM] D6's adopter-first repair is one-sided and its halt note is unconditional

**Location:** `_lyx/discussion/decision-record.md:141`, with Open risks line 212 and the criterion at line 243.
**Issue:** D6 says the local parent holds a squash of a task origin already landed and repairs it by moving the local parent to origin's tip in the parent's worktree.
Fabric's parent-side merge concludes a commit on warp and then on weft, and Finalize's `pushParent` pushes the warp branch only.
After the adopter lands first, the old machine's parent pair therefore also holds a local merge commit on its records branch that the one `git reset --keep` does not touch, and "never a merge" applies to that side equally.
The Decision is silent on it, and Open risks records only the code-side reset.
The halt note is also said to name the reset in every such stop, but a refused parent push has other causes, and a reset to origin's tip on one of them drops a landing nobody else made.
`git reset --keep` is not in the denied list (`PATTERN-no-denied-recovery`), so the form itself is allowed.

### [MEDIUM] The declared PATTERN text is wider than the design it describes

**Location:** `_lyx/discussion/decision-record.md:252`, against D3 lines 55-73, D5 line 117 and Constraints line 193.
**Issue:** The listed entry says starting, resuming and landing are judged against origin inside `ReconcileOrigin`.
D3 judges a start from the Bolt it just pulled, reads the local Bolt for a resume with origin unreachable, and re-reads owners after each push; D5 step 3 re-reads after the push.
Constraints list these reads as exceptions to `PATTERN-origin-reconcile` only, so the departure from the listed `PATTERN-owner-machine` text is not cited against it.
The departures are bounded and recorded in D3 and Open risks, so the repair is to the text and the citation, not to the design.

## Rejected

- D6 carve-out picks one behaviour (the run keeps its lock, finishes Finalize, and the pause holds it afterwards), states what it lets through (the done mark, the parent push, the pull request close) and bounds it to the time Finalize runs; the pause-exemption mechanism is as stated, since `SlugEntryExempt` is `holdsParentLock` (internal/pauserecord/hold.go:63, 271). The old-machine-pushes-first order works: the adopter's holdParent reads a parent that holds the task's work as not moved and takes the already-landed path (internal/landingshed/doc.go). The open points are F1 and F3.
- D3 states one rule over the Spawn seam, and the criterion at line 224 covers every spawn on an existing run. The four call sites of `deps.Spawn` (fresh or unconfirmed spawn, awaiting-child resume, publish retry, binary re-step) are all inside the seam the Decision names; the call-site enumeration is the plan writer's, per the rubric. Not a defect.
- D7's halt messages: each named way forward ends in a resumed run that then passes the owner checks, and the criteria at lines 236, 237 and 244 name a call for each case. The step-1 stops that the plain forms can hit are stated in D7 for the cases where they arise. Not a defect.
- D3's unreachable-origin fallback states what it lets through (two drivers on one run until Finalize's landing reconcile or a later D6 check) and what bounds it, and Open risks records it. The only gap is the PATTERN text, filed as F4.
