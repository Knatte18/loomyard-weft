```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens on the adopt-robust decision record and support log.

**Findings**

1. **BLOCKING** — D12 × D6/D11 (ordering). D12 reads the old owner's archive tags right after the `reconcile` step. D11's old-owner dobby archives only once its probe reads the new owner, and that happens only after D6's `owner` step (step 4) pushes. So in the normal takeover the D12 read runs before the old owner can have archived anything for this adopt. On a resume (D2) the reconcile is not re-run, so the read never sees the archives at all. D12's purpose ("the new owner sees what the old owner had not pushed") goes unmet except for archives left by an earlier takeover. Evidence: D3's step order puts reconcile at 1 and owner at 4, and D11 is triggered by "a dobby's probe reads its subject owned by another machine".

2. **MEDIUM** — D4 × D11 (same dobby foreign-owner path). On the adopting machine, between `reconcile` and `owner`, the local dobby reads the old owner as foreign. D4 says only that it "does not exit" and starts nothing. D11's steps before the exit (archive this machine's local tips, owner-pause each run it drives) are attached to the same read, and D4 does not say it suppresses them. So the adopting machine may push archive tags for its own tips and owner-pause, interrupting sessions, on runs D5 then converts. D4 should state that an unfinished record suppresses D11's whole sequence, not only the exit.

3. **MEDIUM** — D10 × D2 (the refusal blocks `lym s`). Take an interrupted or stopped record for `root`, then `lym s` in a beam: `adopt --here` resolves to the beam, a different subject, so D2 refuses. The shortcut stops at that non-zero exit (D10 relies on this), so `beam start` never runs, even when this machine already owns the beam. D2 also never says whether `--here` is matched as its resolved subject or as a request form. The record should state the subject match for `--here`, and say whether an owned-here `--here` refuses or passes through when another subject's record stands.

4. **MEDIUM** — D1 × D2 (race between two callers). With no record standing, two `lym adopt` calls with different subjects both pass the start check and both spawn a worker. One worker wins `adopt.lock` and the other exits writing nothing. The losing caller then follows a record of other subjects and, per D1, prints that adopt's envelope and exit code as its own. D2's other-subjects refusal is defined only "when `lym adopt` starts", not after the loser's worker exits. The record should say the caller compares the record's subjects once its worker settles, and refuses on a mismatch.

5. **MEDIUM** — D2 × D7/D10 (flag merge on resume). "Taking the call's flags over the record's" is ambiguous for boolean flags the call omits. If omission counts as false, an adopt that was interrupted under `--no-orch` (from an orch strand) and is resumed by a plain `lym adopt <subjects>` from a shell starts the orch, against D7's intent. The record should state that resume ORs the flags, or keeps the record's flags unless the call overrides them.

6. **LOW** — D5 × D4 (acceptance). The D5 criterion says "`lym tender resume <slug>` lifts it, and the dobby then starts the full pass". While an interrupted or stopped record names the subject, D4 keeps the dobby idle, so no pass starts even after the lift. The criterion and D5's text should say "once no unfinished record names the subject", or D9's way forward should name `--discard` for this case.

7. **LOW** — D4 × D8/red check (interrupted record). If the worker dies after the `check` step starts the detached check but before `done`, a red check files the fix task. D4 then keeps the dobby from starting it until a resume or `--discard`. Benign, but unstated.

8. **LOW** — D4 acceptance × D11. "With no record … the watcher and the dobby exit on a foreign owner as today" conflicts with D11, under which the dobby first archives, owner-pauses, and stays up when the archive fails. The criterion should name D11's behaviour for the dobby.

**Pairs checked**
- D1×D2 (start, spawn, lock, follow): broken (finding 4)
- D1×D10 (non-zero exit at the follow cap stops the shortcut): held
- D2×D3 (resume at the first step not done; every step safe to re-run): held
- D2×D7 (flag merge vs `--no-orch`): broken (finding 5)
- D2×D10 (`--here` subject match, refusal of other subjects): broken (finding 3)
- D2×D12 (resume skips the reconcile, so no archive read): broken (finding 1)
- D3×D6 (owner before deploy in the step list): held
- D4×D5 (window between reconcile and pause covered by the record): held
- D4×D5 (operator resume while a record is unfinished): partially broken (finding 6)
- D4×D6 (suppression lasts through the deploy until `done`): held
- D4×D7 (BeamUp leaves a live dobby alone, which stays idle until `done`): held
- D4×D8 (green end runs after `done`, lifts nothing): held
- D4×D10 (`--here` reconcile is covered by the record's subject): held
- D4×D11 (dobby foreign-owner path): broken (finding 2)
- D4×red check (fix task during an interrupted record): bound unstated (finding 7)
- D5×D7 (dobby started with every adopted run paused; live-pair runs unpaused, as today): held
- D5×D8 (green no longer lifts, legacy hold reads the same): held
- D5×D11 (`OwnerCaller` from the old owner turned into `AdoptCaller` in place on a later adopt): held
- D6×D11 (owner push triggers the old-owner archive): held, but feeds finding 1
- D7×D10 (`--no-orch` in the shortcut, then `beam start` starts the orch): held
- D9×D10 (owned-here `done` record shows nothing): held
- D11×D12 (archive timing vs read timing): broken (finding 1)
- Constraint "each step safe to run again" × D3, D5, D6, D7: held
- Acceptance D4 × D11: broken (finding 8)
- Acceptance D5 × D4: broken (finding 6)

I made no further tool calls.
