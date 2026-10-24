```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens. For each changed function, I ran one concrete case and judged the result.

**Cases run**

| Function or branch | Case | Result |
|---|---|---|
| `battencli.ResolveBattenPrime` | Prime location | Correct: returns the location. |
| | Beam pair with an origin record | Correct: returns the prime. |
| | Beam whose entry holds a run status | Correct: refused, naming the landing and the parent worktree. |
| | Task pair with no origin record | Correct: refused, naming `nearestBeamAbove`. |
| `nearestBeamAbove` | Parent gone, record unreadable, or a cycle | Correct: ends at the prime. |
| `refuseOwnPair` | Acting is the slug | Correct: refused. |
| | Acting is the prime | Correct: passes. |
| | `cancel --remove` | Correct: also wired, in `cancelcmd.go:108`. |
| `hubgeom.ParentOf` | No origin record | Correct: `ParentBeam` decides. |
| | Origin with empty `ParentWorktree` | Correct: prime, never the board. |
| | `ParentBranch` or `ParentWorktree` matches the prime | Correct: prime. |
| | Beam pair absent here | Correct: the recorded `ParentWorktree` is returned with no local-pair match. |
| `NearestLiveOrch` and `nearestLive` | `from` is the prime | Correct: start is `""`, so the prime's orch name. |
| | Parent gone, record absent, or a cycle | Correct: ends at the prime. |
| | Parent is the prime | Correct: `""`. |
| `beamForkParent` and `beamForkPrecondition` | Beam pair absent, branch on origin or not | Correct: names `lyx adopt` or `lyx beam create`. |
| | Beam run-held by its landing | Correct: refused. |
| | `ParentBeam` empty | Correct: prime. |
| `startGuard.refuse`, `refuseProject`, `forksFromPrime` | Grouping project | Correct: refused, naming `lyx beam create`. |
| | Beam with a marked pair | Correct: skips the dirt and left-pair checks. |
| | Member forking from a beam | Correct: skips the dirt check. |
| | Beam resumed past Worktree-Create | Correct: passes. |
| | Task whose pair carries the beam mark | Correct: refused, naming `lyx beam promote`. |
| `windowSite` | Parent empty, or beam `.git` absent | Correct: prime. |
| | Beam present | Correct: the beam, with `ensureSession` true. |
| `AddOptions` and `Add` | `Parent` set | Correct: refusals come before any mutation, dirt is logged not refused, the fork uses `parent.headSHA`. |
| | `Parent` empty | Correct: the dirt refusal is kept. |
| | `AdoptLocal` on an unmarked branch | Correct: adopted. |
| | `AdoptLocal` on the default branch | Correct: records an empty parent. |
| | `AdoptLocal` on a branch checked out elsewhere | Correct: refused. |
| | `AdoptLocal` over an adopted weft that already has an origin record | **Incorrect**: see finding 1. |
| `endRunPhase`, `cancelHold`, `clearRunPhase` | `TakeMarker` fails | Correct: the deferred `hold.end` restores the gate. |
| | Windows-live refusal | Correct: `markerDone` stops a double release. |
| | EndOnly | Correct. |
| | Promote clear with `hold == nil` | Correct: guarded. |
| `beamcli.create` and `judgeCreateEntry` | Absent entry, note, task, done project | Correct: each refused with its way forward. |
| | Beam owned elsewhere | Correct: refused. |
| | Beam with owner none | Correct: passes. |
| `createSteps` | Owner written, then `Add` fails | Correct: re-run resumes. |
| | Pair present | Correct: `MarkBeam` instead of `Add`. |
| | Push | Correct: repeated on every run. |
| `promote`, `judgePromoteTask`, `promoteSteps` | Re-run after step 2, 3 or 4 | Correct: the owner is kept by `clearRunClaim`, which clears the status only. |
| | Run past the parent-side merge | Correct: the cancel's `ChildReachedParent` refusal covers it. |
| | Root pair | Correct: refused. |
| | Open dependents, no pair, owned elsewhere | Correct: each refused. |
| `promoteProject` | Beam with a marked pair | Correct: workspace step only. |
| `land` and `judgeLandEntry` | Acting equals the slug | Correct: refused. |
| | Task, grouping project, no pair, unmarked pair | Correct: each refused. |
| | Envelope `ok` | Not verified: the `envelope["ok"]` contract of `ResumeInWindow` is outside what I read. |

**Findings**

1. **LOW, design.** `internal/fabricengine/add.go:506` (the origin-write condition). The origin record is written only when `!weftAdopted || statErr != nil`. If `lyx beam create` runs `Add{Parent, Beam: true}` over a weft branch that is adopted and already carries an origin record, the record is kept as it was and the beam mark is silently not written. `create` reports `PairAdded` and `BeamMarked` false. It regenerates the workspace files without the beam file and the beam never reads as a beam. The next `create` re-run repairs it through `MarkBeam`. The same applies to a new `Parent` or `ParentBranch`, which are not applied to an adopted record. D9 step 2 requires the mark "set in the same origin commit". The fix is to set `Beam` on an existing record when `opts.Beam` is true, or to have `create` call `MarkBeam` after a `PairAdded` result.

Nothing else wrong in the rest of the lens.

**Tool calls**
1. `grep ClearClaim` and `sed 225,255 cancel.go`: whether `endRunPhase` can return a nil `hold` on success, and where the claim clear is wired. My context lacked the middle of `endRunPhase`.
2. `sed 280,300 cancelcmd.go` and `grep` for the clear-claim function: whether clearing the claim also clears the owner, which decides whether a promote re-run after step 3 trips the "never started" refusal. My context lacked it.
