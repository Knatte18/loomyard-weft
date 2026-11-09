# Round 2 fixer report — discussion decision record

Every finding's premise held against the code; all twelve are fixed in `_lym/discussion/decision-record.md`, and the support log's Review rounds gains a Round 2 entry.

## Changes

- F1: fixed differently. `machinename.Names` (D3) and the verb's rewrite (D5) drop an empty `names` entry, D1's former-name definition says non-empty, and criteria pin both.
  Not made: dropping a non-empty entry that fails D4. Such an entry matches only an owner recorded under that exact string, which only this machine could have written; the hostname rule's output is not checked against D4 (`internal/machinename/machinename.go:23-24` keeps `_`), so after a hostname change dropping it would strand owners and landings under an old hostname-rule name (self-check, fix-proposal 2). The harm F1 cites is the empty owner read as no owner (`internal/boardengine/owners.go:175-177`).
- F2: fixed as proposed. D5 counts the hostname rule's name as this machine's in the collision check and appends it to `names` on every write, a re-run and both repair branches included; the undecodable-file sentence now follows the general rule; criterion added.
  The self-check added the shared short hostname case to D5's shared-name bound.
- F3: fixed as proposed. D3 makes every `machinename` error carry its own way forward and forbids wrappers adding a hostname fix; criterion added.
- F4: fixed as proposed. D3 gains the bound on the dobby's `beamOwnedElsewhere` and batten's refusal.
- F5: fixed as proposed. D5's shared-name bound names a live machine with no recorded owners and covers all cases.
- F6: fixed as proposed, and the self-check added that a lost entry under another name stays that machine's, taken back with `lym adopt`.
- F7: fixed as proposed in D6's first recovery bullet.
- F8: fixed with the second option: the Goal states the relative `HOME`/`LOCALAPPDATA` exception. D3 keeps the error, since returning the hostname name when the path cannot resolve is the silent wrong name D3 refuses.
- F9: fixed as proposed in D1.
- F10: fixed as proposed.
- F11: fixed as proposed: the background-change bullets moved into the two PATTERN Constraints bullets; Notes keeps the overview-lines hint and the sole-source scan hint.
- F12: fixed as proposed in D8, the bound now "off Windows".

## Self-check

- fix-claims: no findings.
- fix-consistency 1, LOW (D4's no-change sentence): fixed as proposed, narrowed to "the rule refuses no machine's hostname-rule name".
- fix-consistency 2, LOW (collision criterion says former name): fixed as proposed.
- fix-consistency 3, LOW (repair branch and criterion keep `names` as is): fixed as proposed.
- fix-consistency outside-lens note (another machine's owners under a shared hostname-rule name): merged with fix-proposal 1.
- fix-proposal 1, MEDIUM (shared short hostname swept): fixed with own change: D5's shared-name bound names the case with the existing way forward. Not made: dropping the always-count rule, since a shared hostname-rule name already made both machines read each other's owners as their own before any file existed, and without the rule F2's stranding returns.
- fix-proposal 2, LOW (invalid-entry drop strands an old hostname name): fixed as proposed, see F1.
- fix-way-forward 1, LOW (no way forward for a lost entry under another name): fixed as proposed in D6.
- fix-way-forward 2, LOW (push-only re-run reported as nothing to sweep): fixed with own change: D6 has the verb read before the write whether the hub's Bolt is ahead of origin and report a push-only re-run as pushed. `SetOwners` pushes with no new commit (`internal/boardengine/ownerset.go:131`), so the re-run does push.

## Disputed

None.

## Deferred

None.

## Rulings

None.
