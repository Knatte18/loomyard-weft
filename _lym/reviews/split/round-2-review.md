---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/raddle/proposal.yaml:91
    summary: transition-push-takeover carries three criteria whose behavior a later part builds, so it can pass them only by building that part's work or by a production writer that only its tests call
    fix: >-
      In _lym/raddle/proposal.yaml, move three whole criterion bullets out of transition-push-takeover's criteria.
      Move the bullet starting "D1: a run claim, a landing's board `done`, an owner write and a hold write are on origin" to machine-registry-holds-catchup's criteria,
      since that part's boardengine setter writes the hold.
      Move the bullet starting "D2: an adopter's takeover push rejected over the old owner's later transition" to adopt-foreground's criteria,
      since that part delivers the takeover entry's production writer and its rejected-push retry.
      Move the bullet starting "D2: after an in-loop hand-over no local pair of the slug stands" to leftover-pairs-teardown's criteria,
      since that part delivers the dobby's local-mode removal of a pair a hand-over mark names.
      Each target part has transition-push-takeover on its depends_on path, so the carried D1 and D2 ids stay built.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/raddle/proposal.yaml:146
    summary: the upgrade's first read in machine-registry-holds-catchup needs adoptcli's fix-tries reader, which its takes does not list, while adopt-foreground's takes claims that reader as the earlier part's delivery; neither body fixes where the first read runs or how it reaches the reader
    fix: >-
      In _lym/raddle/proposal.yaml, name the interface in both parts' bodies.
      In machine-registry-holds-catchup, add to contract.takes the fix-tries.json count reader in internal/adoptcli/holds.go (today the unexported readTries),
      and add to contract.delivers the package the upgrade's first read runs in and the exported reader it calls,
      placed so no import cycle forms (internal/adoptcli imports internal/dobbyengine and internal/pauserecord, so a first read in either cannot import internal/adoptcli; move the reader to a package both can import, or run the first read from internal/adoptcli's caller side).
      Add that exported reader to machine-registry-holds-catchup's contract.exports with the first read as its production caller.
      In adopt-foreground, make the takes and delivers mentions of the fix-tries reader name the same package and function.
---

### [BLOCKING] transition-push-takeover carries criteria a later part builds

**Location:** `_lym/raddle/proposal.yaml:91`, `:95`, `:97`

**Issue:** Three of the part's criteria test behavior that its own contract leaves to a later part on the `depends_on` chain.

- `:91` asks that "a hold write" be on origin when its verb returns.
  Holds and their setters arrive in `machine-registry-holds-catchup` (its `delivers`: "board.json's machines map and holds list written only by boardengine's setters with a synchronous push").
  No hold or hold writer exists when `transition-push-takeover` runs.
- `:97` asks that "an adopter's takeover push rejected over the old owner's later transition writes the takeover entry onto origin's tip and lands".
  D2's last bullet makes that retry re-read the adopter's hold and the subjects' owner from Bolt.
  `adopt-foreground` delivers "the takeover entry's production writer and its rejected-push retry", and `transition-push-takeover` exports only the entry type, its reader and a test writer in `internal/testkit`.
  To pass this criterion the first part would have to add a production writer and retry with no production caller in the part, against `PATTERN-test-only-code`, or build D6's retry without holds.
- `:95` asks that, for a caller inside the pair, no local pair stands "after the parent dobby's next pass".
  D2 leaves that removal to the parent's dobby under D7.
  `leftover-pairs-teardown` delivers "the dobby removing in local mode the pair of a run its hand-over mark names while another machine owns it", and `transition-push-takeover` lists no `internal/dobbyengine`.
  The catch-up in `machine-registry-holds-catchup` runs only after an absence, not on each pass, so it does not cover this either.

The gate's `carried-decision-unbuilt` check reads decision ids only, and D1 and D2 are built by this part, so the gate passes all three bullets.
The seam does not hold: the part's criteria take what no part before it delivers.

### [MEDIUM] The upgrade's first read has no fixed route to the fix-tries reader

**Location:** `_lym/raddle/proposal.yaml:146`, `:147`, `:198`, `:199`

**Issue:** D5 has the upgrade's first read read each subject's try count from `internal/adoptcli`'s `fix-tries.json` when it writes an `adopt-fix` hold.
`machine-registry-holds-catchup` builds that first read but its `takes` does not list the reader.
`adopt-foreground`'s `takes` lists "the fix-tries.json reader in internal/adoptcli/holds.go its upgrade first read calls" as a `machine-registry-holds-catchup` item, and its `delivers` keeps that reader "staying for the upgrade's first read".
The two parts disagree on who exposes the reader.
In the code, the reader is the unexported `readTries` (`internal/adoptcli/holds.go:21`).
`internal/adoptcli` imports `internal/dobbyengine`, `internal/pauserecord` and `internal/boardengine`, and only `internal/beamcli` and `cmd/lym` import `internal/adoptcli`.
A first read placed in the dobby start or in `pauserecord` cannot call it without an import cycle.
Neither part's body says where the first read runs or what it calls, so the first part guesses at an interface the second part also claims.

## Rejected

- Focus 1, hold setters: not a defect.
  The one hold write sits in `machine-registry-holds-catchup` with the upgrade's first read as its caller.
  The rewrite, adopt-fix replacement, try-count update and clear setters sit in `adopt-foreground` "beside their callers here", exported with "each with its production caller in adopt".
  The registry setter's callers (catch-up, dobby start, owner write, `lym machine name`, `--forget-former`) are all in the same part.
- Focus 1, hand-over mark drop: not a defect.
  `adopt-foreground` adds the drop function beside the mark reader and calls it from step 3's takeover write, in the same part.
  It lists `internal/fabricengine`, and its `takes` names the mark's path and reader from `transition-push-takeover`.
- Focus 1, other exports: not a defect.
  The takeover reader has its caller in the rejected-push path, its test writer sits in `internal/testkit`, the catch-up entry point is called by the dobby start in the same part, the pause-and-confirm function by hibernate's carrier and the stop, `internal/desktoplauncher`'s add and remove by the dobby start, catch-up and verb, and the orphan grace and first-sight path by the dobby.
  The one export that a test needs in a part with no production caller is the hold clear, which `machine-registry-holds-catchup`'s D5 criterion "drops only those entries once the hold clears" exercises.
  Its tests can clear the hold through a `testkit` writer, which `PATTERN-test-only-code` admits, so this is no finding.
- Focus 2, D2's mark drop and D6 step 1's one commit: not a defect.
  Both are named in `adopt-foreground`'s `delivers`, and its `takes` lists the mark reader, the hold types and readers, the registry registration write with D3's refusals and `boardengine.SetOwners`'s pull-first commit and rejected-push re-read (`internal/boardengine/ownerset.go:115`).
- Focus 3, chain: not a defect.
  Every part listing `internal/boardengine` is on one chain: `transition-push-takeover` → `machine-registry-holds-catchup` → `adopt-foreground` → `board-prints-root-beam`.
  The record gives D8 no ordering reason beyond the shared package, and placing it last lets it run beside `leftover-pairs-teardown`, `hub-stop-all` and `machine-start-launchers`;
  placing it first would let it run beside nothing.
  `hub-stop-all` and `machine-start-launchers` share no package and run side by side; each shares a package with `leftover-pairs-teardown` (`internal/beamcli`, `internal/dobbyengine`), so that edge is forced.
- `transition-push-takeover`'s criterion that a rename on an adopting machine hands nothing over is buildable in that part: `machinename.Names` already returns the current and former names of `machine.json` (`internal/machinename/machinename.go:83`).
- Cost lines: every darn figure matches its fit and its summed package sizes, and the total of $28.25 matches the four darn parts.
