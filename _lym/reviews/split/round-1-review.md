---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/raddle/proposal.yaml:148
    summary: machine-registry-holds-catchup exports hold setters (rewrite to another machine, replace with adopt-fix, try-count update, clear) whose only production caller is adopt-foreground, so the part lands functions only its tests call and fails PATTERN-test-only-code's guard
    fix: In machine-registry-holds-catchup, keep only the hold write its upgrade first read calls (an adopt or adopt-fix hold) and the hold readers; drop the rewrite, replace-with-adopt-fix, try-count update and clear setters from its exports and delivers. In adopt-foreground, add internal/boardengine to packages, replace no boardengine change in delivers with building those setters in boardengine beside their callers, and drop them from its takes, so each new setter has its production caller in the part that adds it.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lym/raddle/proposal.yaml:79
    summary: D2's drop of this machine's hand-over marks on an adopt's takeover entry is built by no part; transition-push-takeover exports a drop function with no production caller in that part, and adopt-foreground neither takes it nor delivers calling it
    fix: In transition-push-takeover's exports, remove the hand-over mark's drop function (keep its path and reader). In adopt-foreground, add to takes the hand-over mark's path and reader from transition-push-takeover, and add to delivers that step 3's takeover write drops this machine's hand-over marks of each moved slug, through a drop function this part adds beside the mark reader.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/raddle/proposal.yaml:81
    summary: board-prints-root-beam heads the depends_on chain, so the whole build waits on it, though the record gives D8 no order and a later place in the boardengine chain lets it run beside three or four other parts
    fix: Set transition-push-takeover's depends_on to [] and board-prints-root-beam's depends_on to the last part of the chain that lists internal/boardengine (machine-registry-holds-catchup as the proposal stands, adopt-foreground once F1's fix gives it internal/boardengine), so board-prints-root-beam runs beside the later parts that touch none of its packages.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/raddle/proposal.yaml:198
    summary: D6 step 1 writes the adopt hold, the subjects' owner and the registry entry in one Bolt commit with a rejected-push re-read, but the exported setters are separate writes and boardengine.SetOwners makes its own commit, while adopt-foreground promises no boardengine change
    fix: Name in the boardengine contract of whichever part builds the adopt's hold setters (adopt-foreground after F1's fix) one pull-first Bolt write that records the adopt hold, or under --take-hold the rewritten hold, together with the subjects' owner and this machine's registry entry in one commit, and that on a rejected push re-reads Bolt and refuses on an owner of these subjects or a hold another machine wrote since; add it to that part's delivers and to the takes of any part that calls it.
  - id: F5
    severity: LOW
    class: scope
    location: _lym/raddle/proposal.yaml:198
    summary: adopt-foreground retires adopt's local state and may remove internal/adoptcli's package, but its contract keeps no fix-tries.json reader, which machine-registry-holds-catchup's upgrade first read calls for the adopt-fix hold's try counts
    fix: In adopt-foreground, add to takes the fix-tries.json reader in internal/adoptcli/holds.go that machine-registry-holds-catchup's upgrade first read calls, and add to delivers that this reader stays for the upgrade's first read.
  - id: F6
    severity: LOW
    class: scope
    location: _lym/raddle/proposal.yaml:305
    summary: D10 removes a hub's launcher when the verb, the dobby or any catch-up finds this machine owns nothing there, but machine-start-launchers delivers the removal only at the dobby start and by the verb, missing the catch-up after a sleep or a hand pass
    fix: In machine-start-launchers' delivers, replace the launcher added and removed at the dobby start beside the catch-up with the launcher added at the dobby start and removed by every catch-up, the dobby start's included, that finds this machine owns nothing in the hub.
---

### [BLOCKING] Hold setters ship with no production caller

**Location:** `_lym/raddle/proposal.yaml:148`

**Issue:** `machine-registry-holds-catchup` exports five hold setters "the foreground adopt calls", so that `adopt-foreground` "changes no boardengine code".
Inside that part, only the upgrade's first read writes a hold (D5), and it only writes an `adopt` or `adopt-fix` hold.
Rewriting a hold to name another machine (`--take-hold`), replacing it with an `adopt-fix` hold, updating try counts and clearing it are D6's steps, and their only callers are in `adopt-foreground`.
`PATTERN-test-only-code` fails any production function no `main` reaches, and `cmd/lym/testonlycode_linux_test.go` enforces it.
The part would land four setters that only its tests call, so its gate fails.
The alternative is that its implementer adds callers D3 to D5 never asked for.
The record's constraint puts "the hold setter in the upgrade's first read" in this part, and that covers the write alone.

### [BLOCKING] The hand-over mark's drop is built by no part

**Location:** `_lym/raddle/proposal.yaml:79`

**Issue:** D2 says an adopt's takeover entry naming this machine (D6 step 3) drops this machine's hand-over marks of that slug.
`transition-push-takeover` exports "the hand-over mark's path in internal/lymdirs and its reader and drop function".
Nothing in that part calls the drop, since the takeover's production writer belongs to `adopt-foreground`, so the function breaks `PATTERN-test-only-code`.
That part's own takes line also says every new function has a production caller in the part.
`adopt-foreground` takes "the takeover entry type, reader and synchronous pushes" and delivers "the takeover entry's production writer and its rejected-push retry", but neither names the mark or its drop.
Neither part's contract builds this piece of D2.
`leftover-pairs-teardown`'s criterion that a re-adopted run keeps its pair rests on that drop.

### [MEDIUM] The root-beam part lengthens the chain for no ordering reason

**Location:** `_lym/raddle/proposal.yaml:81`

**Issue:** The chain runs `board-prints-root-beam` → `transition-push-takeover` → `machine-registry-holds-catchup` → `adopt-foreground` → `leftover-pairs-teardown` → {`hub-stop-all`, `machine-start-launchers`}.
That is six parts deep, and the only unordered pair is the last two.
The record states no order for D8.
Its Notes say only that D8 shares `boardengine` with D3 and D5 and no code with the rest.
So `board-prints-root-beam` must be chained to the parts listing `internal/boardengine`, but it need not go first.
Chained after `machine-registry-holds-catchup`, it lists no package shared with `adopt-foreground`, `leftover-pairs-teardown`, `hub-stop-all` or `machine-start-launchers`.
It then runs beside all four, and the chain drops to five parts.
After F1's fix gives `adopt-foreground` `internal/boardengine`, chaining it after `adopt-foreground` still runs it beside the last three.
The cut keeps the part count and runs fewer parts at once than that alternative.
The stencil's rule to put a part that waits for nothing first in its chain does not reach D8: the record states no order for it.
Nor does any later part gain from it.

### [MEDIUM] D6 step 1's single Bolt commit has no setter that builds it

**Location:** `_lym/raddle/proposal.yaml:198`

**Issue:** D6 step 1 writes the `adopt` hold, this machine as the subjects' owner and its registry entry in one Bolt commit.
A rejected push then re-reads Bolt and refuses on an owner or a hold another machine wrote since.
`--take-hold`'s commit likewise rewrites the hold and writes the owner together.
`machine-registry-holds-catchup` exports separate hold setters, each pushing synchronously, and the owner write stays `boardengine.SetOwners`, which makes its own pull-first commit (`internal/boardengine/ownerset.go:115`).
`adopt-foreground` delivers "no boardengine change", so it can only chain two commits.
Two commits split the hold from the owner, which the criterion "two machines adopting at once end with one owner; the other's adopt refuses naming the winner" depends on.
The seam between the setters one part exports and the step another part builds does not hold.

### [LOW] The upgrade's try-count reader is not kept by the part that rewrites adopt

**Location:** `_lym/raddle/proposal.yaml:198`

**Issue:** D5's upgrade first read takes each subject's try count from `fix-tries.json`, read today in `internal/adoptcli/holds.go`.
`machine-registry-holds-catchup` builds that read and lists `internal/adoptcli`.
`adopt-foreground` then moves the try count into the `adopt-fix` hold and retires adopt's local state.
Following D6, it may also remove `internal/adoptcli`'s package with the record.
Its contract neither takes nor keeps the reader the earlier part's first read calls.
That leaves the implementer free to delete it with the rest of adopt's local state.

### [LOW] Launcher removal misses the catch-up after a sleep

**Location:** `_lym/raddle/proposal.yaml:305`

**Issue:** D10 removes a hub's launcher when "this machine's verb, dobby or catch-up (D4) finds that this machine owns nothing in the hub".
A catch-up runs after a sleep and on a hand pass as well as at the dobby start.
`machine-start-launchers` delivers the launcher "added and removed at the dobby start beside the catch-up ... and by the verb", which omits the removal at the other catch-ups.

## Rejected

- `leftover-pairs-teardown` after `adopt-foreground`: both list `internal/dobbyengine`, `internal/beamcli`, `internal/fabricengine` and `internal/battencli`, so they must be chained.
  D7's re-adopt criterion needs adopt's takeover writer, so this order follows the work; it is no order departure, since the record orders neither before the other.
- `hub-stop-all` and `machine-start-launchers` after `leftover-pairs-teardown`: `hub-stop-all` shares `internal/beamcli` with every earlier large part and `internal/orchcli` and `internal/hibernateengine` with some.
  `machine-start-launchers` shares `internal/dobbyengine` and `internal/machinecli`, so both chain behind the last of them.
  The record requires only D1 before D9 and D3 before D10, and the extra links come from shared packages, not from a choice.
- `hub-stop-all` beside `machine-start-launchers`: their packages do not overlap.
  The restart drives `lym beam start` as a child process, which the stop leaves unchanged, and `orchcli.BeamDown`'s added Claude-process end is the stop's alone, so neither guesses at the other's interface.
- Merging D1 with D2 and D3 to D5: each pair shares `internal/fabricengine` or `internal/boardengine` and `internal/dobbyengine` heavily, so a cut along each decision would chain the parts it adds and run nothing more at once.
- Recipes and cost lines: the darn cells read from `lym report cost` match each cited fit, and each cost line's line count matches its packages' sizes.
  These are darn solid stored-record at $6.88 fixed and $0.23 per 100 lines over 15 runs, darn light stored-record at $2.49 and $0.52 over 5 runs, and tapestry solid stored-record at too few runs.
  `machine-registry-holds-catchup`'s sizes sum to 2570 against the stated 2580, which changes no figure since its cell reads too few runs.
- The Added after Discussion entries: `hub-stop-all` delivers no tracked record, no per-run intent and a standing hold left as it is.
  `board-prints-root-beam` keeps `boardengine.ParentBeam`, `Board.ParentBeamOf` and `hubgeom.ParentOf` returning the empty slug.
  The arbiter's entry changes no decision.
