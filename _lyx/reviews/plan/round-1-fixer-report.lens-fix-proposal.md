```yaml
lens: fix-proposal
outcome: reported
```

Report: fix-proposal lens, challenging the reviewer's proposals the fixer carried out as proposed.

**MEDIUM: F7's `DobbyLive` re-entry check can post a false "interrupted landing" notice on a fresh run.**
- **Changed text:** `_lyx/plan/15-pawl-engine.md`, the Beam-Await bullet. It says that on any entry with no request standing, the row reads `DobbyLive` and, when the dobby is not live, posts the interrupted-landing notice naming `lyx beam start &lt;slug&gt;` and `lyx beam land &lt;slug&gt;`. Card 17 wires the seam as `dobbyengine.Live`.
- **Case:** Beam-Up's `Steps` (`orchcli.BeamUp` → `EnsureLive`) spawn the beam's dobby detached and never wait for it. Beam-Await is entered right after, with no request standing. `dobbyengine.Live` tries the dobby's lock without blocking (`internal/dobbyengine/start.go:179-186`), and a child that has not yet taken its lock reads not live. So the first ordinary entry of a new pawl run can tell the orch that a landing was interrupted when none happened.
- **Second case:** `BeamUp` only records a failed dobby start in its report and does not halt. That case gets the same "interrupted landing" wording, which is wrong; the dobby simply never started.
- **What would fix it:** confine the notice to re-entry, not the first entry after Beam-Up. Alternatively, word it as "the beam's dobby is down", and give the first read a grace period or retry it on the next wake.

**LOW: F8's held-branch wording in card 20 puts a cancel in front of every member blocker.**
- **Changed text:** `_lyx/plan/20-batten-cancel-beam.md`, the `refuseBeamCancel` sentence. It says each member blocker is worded "as card 14's `landHaltReason` does … `lyx batten cancel &lt;member&gt; --remove` …, then the `lyx board upsert` move".
- **Case:** card 14's `landHaltReason` (as just fixed) names the board move alone for a member with no run status, and the cancel only for a member with a pair or a run in flight. Card 20's sentence names the cancel for every member. A draft member with no run would get a cancel of nothing as its first step.
- **What would fix it:** say "the cancel for a member with a pair or a run in flight, then the board move; the board move alone for a member with no run status".

No other proposal broke under a concrete case:
- **F2:** `hubgeom.ParentOf` falls back to the board when no origin record exists (`internal/hubgeom/parentof.go:26-31`). So reading `AdoptSubject` in the absent-worktree branch of `judgeOrigin` does not turn a working cancel into a probe failure.
- **F14:** I found no caller that names an unpaired beam.
- **F15:** card 11 resolves `root` to the prime, so `lyx beam start root` works.
- **F1, F4, F5, F6, F9–F13, F16, F17:** I found no breaking case.

Tool calls:
1. Read `hubgeom.ParentOf` and `dobbyengine`'s `ensureOne`/`Live`, to test F2's resolver with no worktree and F7's liveness read right after a spawn. My context held neither function body.
