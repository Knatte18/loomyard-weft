```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of the record's decisions that act on the same state for whether one undoes or contradicts the other.

**Findings**

1. **MEDIUM — D7 against D1 (`lym tender resume` pushes no pair).**
   - D7 drops the pair push from resume "since D1 leaves no deferred push for it to pay". D1 keeps the anchor-change push and tender's prime push detached, and keeps fabric's pause deferral for them.
   - Resume's `spawnOwed` is what pays a push a pause deferred. Without its `pair:<slug>` spawn, an anchor move deferred under a pause never reaches origin.
   - Evidence: `internal/tenderengine/resume.go` `spawnOwed` ("spawns the detached pushes … that a pause deferred and the lift uncovers"), and D1's bullet "it stays for the detached pushes below".
   - Fix: keep resume's spawn for the deferred detached pushes D1 keeps, and drop only the transition push.

2. **MEDIUM — D10 against D3 and `dobbyengine.OwnedWorktrees` (skipping a whole hub).**
   - D10 skips a hub whose prime another machine owns, or that has no recorded prime owner. A beam this machine owns under such a hub is then never started, and its launcher is removed ("hub owned by another machine").
   - `lym beam start --all` starts owned beams whatever the prime's owner. D10's own rule "starts only what this machine owns" and the entry body's per-item skip ("skips every one another machine owns") both call for starting those beams.
   - Fix: filter per worktree, not per hub. Skip only the prime when no owner is recorded, and keep a hub's launcher while any beam in it is owned here.

3. **LOW — D9's criterion against D5 (`hold` entries).**
   - D9's acceptance bullet says the stop "leaves no pause entry". Under a standing hold, D5's `hold` slug entries stay in the pause record, and D9's second bullet expects the hold to stay. The two criteria contradict each other in that case.
   - Fix: write "leaves no pause entry of its own".

4. **LOW — D9 step 1 against D2 (Finalize cut by the stop).**
   - The stop goes on while a land lock is held, so it can cut a Finalize between the local merge and the parent push.
   - D2 covers that cut only for a takeover. The same machine's next Finalize pass after a restart finds its local parent branch ahead of origin, and nothing in D2 or D9 says that pass pushes it before its transition.
   - Likely held by today's resume, but D9's bound ("nothing pushed is lost") should name this case or cite the existing behaviour.

**Pairs checked**
- D1 × D2 (rejected push reads the takeover before any replay; withheld code side): held.
- D1 × D4 (catch-up pushes a transition's owed local commits after a restart): held.
- D1 × D5 (gate admission against `hold` entries; owed retries held under a hold, the adopter's catch-up runs in adopt's own process): held.
- D1 × D7 (resume's pair push): broken, finding 1.
- D1 × D9 (stop cuts a synchronous push; it stays owed and the catch-up pushes it): held.
- D2 × D6 (takeover writer and reader; step 3 fast-forward; rejected-takeover retry): held.
- D2 × D9 (Finalize cut by the stop): partly unstated, finding 4.
- D3 × D6 (step 1 registers the machine; a clash refuses with D3's way forward): held.
- D3 × D10 (owner filter through the registry via `OwnedWorktrees`): held.
- D4 × D5 (a hold read at the pull applies at once): held.
- D4 × D10 (catch-up removes launchers): held as a mechanism, but inherits finding 2's per-hub rule.
- D5 × D6 (`adopt` callers retired across parts; the upgrade read in D5): held, since the beam lands both together.
- D5 × D7 (no teardown under a hold): held.
- D5 × D9 (stop's hub entry against `hold` entries; hold left standing): held, apart from finding 3's criterion.
- D6 × D9 (an adopt running in a reed pane is killed by the stop and re-runs from tracked state): held.
- D6 × D10 (`--here` `owned_here` against D10's unowned-prime skip): held.
- D7 × D9 (orphan rule after a stop ends every process): held.
- D8 against the rest (prints only; `ParentOf` unchanged): held.
- D9 × hibernate shared function (entries owned by identity): held.

**Constraints:** `PATTERN-dobby-runs`, `PATTERN-child-processes`, `PATTERN-test-only-code` and `PATTERN-refusal-way-forward` each fit the decisions they touch.

**Tool calls**
- Grepped `internal/tenderengine` and `internal/fabricengine/anchorchange.go` for `spawnOwed` and the deferral. My context held only the subagent's summary of resume, not whether it pays pause-deferred pushes.
