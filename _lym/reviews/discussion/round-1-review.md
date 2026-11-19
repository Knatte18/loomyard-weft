---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:124
    summary: "D5 resolves `lym board open` with `lymcwd.ResolveWithin`, which PATTERN-cwd-resolution reserves for a read-only verb; `open` starts a board-writing server and writes a launcher and state, so it is no read-only verb."
    fix: "In D5's first bullet, resolve `open` with `lymcwd.Resolve`, as the other board verbs do; the VS Code task's cwd is the workspace folder, which is the anchored directory. Drop 'so the verb runs from anywhere inside the worktree's anchored directory or its records sibling'. If the operator still wants `open` to run from any subdirectory, add a `## PATTERN changes` bullet `PATTERN-cwd-resolution — changes:` that lets a verb that starts a hub server call `ResolveWithin`, and state that bound in D5."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:218
    summary: "D9 and D11 rest the token's protection on file mode 0600, but Windows, which Scope In covers, ignores Go's permission bits, so on Windows another user can read the state file or the launcher and write the board as the operator."
    fix: "In D9's second and last bullets, state that the state file and the launcher are written 0600 and, on Windows, with an owner-only DACL, as `internal/githubclient/cache_windows.go` already does. Cite `internal/githubclient/doc.go` as the pin for 'Windows ignores Go's permission bits'. Extend the D9 criterion so that on Windows the file's DACL grants only the owner."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:128
    summary: "D5 reads a held lock as proof that the recorded state is the live server's, but D11 has `serve` hold the lock while it builds, binds and only then writes its state, and also while it drains on exit. An `open` in either window hands over a dead or refusing address."
    fix: "In D11's second bullet, have `serve` remove any leftover state file right after it takes the lock, before it builds the board. In D5's second bullet, have the live path wait, bounded as the start path does, until the state answers a request carrying its token. In D11's exit bullet, have the server release its lock before it waits on in-flight requests and wake every waiting long poll at once. Add criteria: an `open` during a starting server's build hands over the new server's address, and an `open` during a drain starts a fresh server."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:298
    summary: "The server builds its board once from the opener's worktree, whose layout the config closures keep reading (`ValidateSlug`, `ParentOf`, `PairExists` in `internal/boardcli/cli.go`). An `open` run from a task pair's folder ties the server to that pair, and once the pair is torn down every create fails while an open page keeps the server alive. D6's claim that every copy of `board open` gives the same result is false."
    fix: "In D5's start bullet and D11's `open` start bullet, start the server with the prime's anchored directory as its working directory, taken through `lymcwd.NamedWorktree` for the prime, whatever worktree `open` ran from. Replace D11's bound 'A server whose starting worktree is removed keeps serving the hub until it exits' with that rule. Keep D6's last bound sentence only once it holds. Add a criterion: `open` run from a task pair starts a server whose working directory is the prime's anchor."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:226
    summary: "One embedded page serves both modes, but the record says neither where its hash-only CSP lives nor how the page picks its mode. A `<meta>` CSP blocks `board-data.js` at file://, and over http the page's `board-data.js` script tag asks for a path no route serves. A missing `window.BOARD` over http would then show D4's `lym board rerender` line."
    fix: "In D9's CSP bullet, state that the `Content-Security-Policy` is a response header only and never a `<meta>` in `board.html`. In D2, state that the page picks its mode by `location.protocol` before it reads `window.BOARD`. In D4's last bullet, limit the `lym board rerender` line to file://."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:79
    summary: "D3's mixed-version bound misses the main case. `board-data.js` carries the render time, so every sync commit from an upgraded machine touches it. An older binary claims no such path and replays each of those commits as a plain cherry-pick, losing the by-entry merge of `board.json`, so its board writes conflict until it upgrades."
    fix: "In D3's bound, add that an older binary replays every commit of a newer one as a plain cherry-pick, because each touches `board-data.js`, and that its concurrent writes then hit the diverged-Bolt refusal until it upgrades. Or drop the render time from `board-data.js`, so a commit touches it only when the projection changes, and state which of the two the design takes."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:264
    summary: "D10 lets the page release a running task to status none in one click, but its bound does not say what follows. A none status makes the task startable for its parent's dobby while the original run still holds its pair."
    fix: "In D10's release bullet, name the statuses release acts from. Either refuse release of a task whose status is a run status (`running · <producer>`), naming the CLI's `set-status` as the way forward, and add a criterion. Or keep it and state in the bound what the dobby does with a released task whose run is still live, with the code that decides it."
  - id: F8
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:61
    summary: "Several Decisions rest on browser, xdg-open or VS Code behaviour stated with no pin: D2's 'a browser blocks fetch from a page opened at file://', D4's 'as a browser's URL parser does', D5's 'xdg-open with no desktop detected runs a browser in the foreground', D6's runTask behaviour for a label several folders carry and for a missing label, and the workspace `tasks` key. D9's hash-only CSP also leaves out that hashes allow no inline event handler or style attribute."
    fix: "For each claim, record the observed result and its pin: the WHATWG URL Standard's basic URL parser steps for D4's trimming; the browsers' file-origin rules for D2; the xdg-utils `xdg-open` script's generic-mode branch, by version, for D5; VS Code's tasks documentation or the task service source, by version, for D6's workspace `tasks` key, the folder runTask picks when several carry a label, and what it shows for a missing label; and CSP Level 3's hash-source rules for D9. In D9, add the constraint the CSP rule implies: the page carries no inline event handler and no style attribute, and sets styles through the CSSOM."
  - id: F9
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md:392
    summary: "The entry's Delivers line names `lym board open` with 'its refusal naming lym board rerender', and D5 drops it. Whether the committed `board.html` and `board-data.js` still earn their place now that editing needs the server is a question about what the task is for, and no Added after Discussion entry or `Decision for board-viewer-build:` note answers it."
    fix: "Raise both questions in the Open risks line to the run's parent: confirm the dropped refusal, and say whether the two committed render outputs stay. Record the answer with `lym loom decision add`."
  - id: F10
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:140
    summary: "D12 lets an orch session run `lym board open`, and D5 waits on an opener that may run a browser in the foreground. Such a wait is unbounded for an agent blocking on it, which PATTERN-daemon-wakeups' keep-warm rule bears on. D5's bound covers only the VS Code task."
    fix: "In D5, either cap the wait on the opener for any caller with a strand name, at the keep-warm resolver's cap for its role, leaving a still-running opener reaped in the background, or refuse `open` in an orch session as D12 refuses it for a run agent. State the chosen bound in D5's bound paragraph and add a criterion."
  - id: F11
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:429
    summary: "Several stated checks and refusals have no criterion: the file route's `t` token (a trust check), D6's two programmer-error refusals (the fabric mark name, and `folders` or `settings` in the builder), D10's page forms, its per-kind and per-status actions and its stale display, and D5's envelope printing the address without the token."
    fix: "Add criteria: `GET /file/<name>` without `t` or with a wrong `t` is refused and serves nothing; a regeneration whose hook returns a member named `lym.generated` or `folders` or `settings` returns an error and writes no workspace file; the D5 envelope's address holds no token. Add a D10 `Review:` criterion for the create and edit forms, the actions offered per kind and status, and the stale refusal showing the fresh entry beside the unsent edit."
  - id: F12
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:254
    summary: "D10's success answer carries the entry's fresh projection, but the record does not say its revision is taken by D8's function over the entry as a reload reads it, and no criterion covers a second edit made with the revision the first edit returned."
    fix: "In D8's first bullet, state that a write's answer computes the revision by the same function over the saved entry as a fresh load and migration reads it. Add the D8 criterion: an edit carrying the revision a successful write returned is saved."
  - id: F13
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:37
    summary: "`RenderToDisk` renders over a copy with `Discussed` cleared for an entry whose decision file is missing. D1 builds the projection in the render path, and D8 hashes the stored entry, so the record leaves open whether such an entry's revision is taken over the copy, which the check never computes, or over the stored entry."
    fix: "In D1, state that each entry's revision is computed over the stored entry before the render's undiscussed copy, and that the projection's decision-file name follows the copy, so no missing file is listed."
  - id: F14
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:292
    summary: "D11's claim that a hub takeover ends the server through `runproc` holds only for a role outside `subjectPassSpared`, and D11 and D13 name no new `runproc` role, which PATTERN-child-processes asks for."
    fix: "In D11's start bullet, name a new `runproc` role for the board server, registered with `runproc.SelfRecord` under `runproc.HubOwner` and kept out of `subjectPassSpared`. List `internal/runproc` in D13 as gaining that role."
  - id: F15
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:205
    summary: "D8's create refusal promises the existing entry's projection, but `checkNewSlugs` also refuses a slug a raddle store holds, where there is no entry to carry."
    fix: "In D8's create bullet, state that a slug refused for any reason other than an existing entry is answered with the board's own refusal message and no entry, as D10's other refusals are."
  - id: F16
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/support-log.md:104
    summary: "The support log still says refusing the server's start closes the way around the task-creation guard, in the ledger's Q7 rationale and in the 'Let the server set or clear the strand name itself' alternative. D9's and D12's bounds say an agent that reads the state file writes as the operator."
    fix: "In `support-log.md`, change Q7's rationale to 'keeps a run agent from starting a server or opening the page; D9 and D12 state that the token still reaches a same-user process', and cut 'and closes that path' from the Rejected alternatives bullet."
  - id: F17
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:447
    summary: "Notes for the plan writer bullets 3 and 4 restate D6's hook inputs and the binary seam, and D8's check placement after the load migrations and before the mutation."
    fix: "Cut from Notes bullet 3 everything after its first sentence. Cut Notes bullet 4 down to 'D8's check goes in `boardCriticalSection` in `internal/boardengine/board.go`.'"
  - id: F18
    severity: NIT
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:90
    summary: "D3's last bound sentence says a change to an unshown field changes `board-data.js`, but D1 puts the render time in the file, so every write changes it whatever field changed."
    fix: "Replace D3's last bound sentence with 'Since `board-data.js` carries the render time, every board write's sync commit carries it.', or cut it if F6's fix drops the render time."
---

### [BLOCKING] `lym board open` uses a cwd resolver reserved for read-only verbs

**Location:** `_lym/discussion/decision-record.md:124` (D5, first bullet)

**Issue:** PATTERN-cwd-resolution says: "Only a read-only verb a session runs from wherever its harness moved the cwd calls it."
Today's two callers fit that rule: `internal/burlercli/validate.go:105` and `internal/shuttlecli/hookpython.go:82`.
`lym board open` is not read-only.
It starts a server that writes the board under the operator's name, and it writes the launcher and, through the server, the state file.
The Sketch adopted `ResolveWithin` when the viewer was a read-only file opener.
The operator's "Wanted next" made the verb a server starter, and the record kept the resolver without declaring a PATTERN change.
That makes it a constraint violation, so the finding is BLOCKING.

### [BLOCKING] The token's file protection does not hold on Windows

**Location:** `_lym/discussion/decision-record.md:218` (D9, and the launcher bullet at line 228)

**Issue:** D9's bound says "the token keeps out other users".
The token keeps them out only while they cannot read the state file and the launcher, and the record protects both by mode 0600 alone.
`internal/githubclient/doc.go:31` records that Windows "ignores Go's permission bits", and `internal/githubclient/cache_windows.go:17-20` installs an owner-only DACL for that reason.
Scope In covers Windows through the `rundll32` opener.
On Windows the trust boundary D9 states is therefore false, which makes this a false premise in a security bound.

### [MEDIUM] The liveness read trusts a state file the server has not written yet, or a server that is draining

**Location:** `_lym/discussion/decision-record.md:128` (D5) against lines 276–280 and 288–291 (D11)

**Issue:** Three lenses reached this independently: interactions F1 and F2, walkthrough F1, and bound-breaker F2.
While a server is starting, D11's order is lock, build, bind, then write the state, so a held lock sits beside the previous server's state file.
A second key press during that window writes a launcher for a dead port.
That port may now belong to another process.
While a server drains on exit, it still holds the lock and refuses new requests.
The first press after an upgrade then fails.
The support log's Turn 4 records the stale-state hand-off as fixed, but the startup window reopens it.

### [MEDIUM] The server is tied to whichever worktree `open` ran from

**Location:** `_lym/discussion/decision-record.md:298` (D11 bound), with D5 line 124 and D6 line 188

**Issue:** `internal/boardcli/cli.go:162-170` builds `ParentOf`, `PairExists` and `ValidateSlug` over the resolved layout.
`ValidateSlug` reloads the fabric config from `ConfigDirs(layout)` on every create.
In a multi-root window VS Code may run the `board open` copy of a task pair's folder, and the server then serves from that pair.
After the pair is torn down, page creates fail.
An open page asks at least once a minute, so the idle exit never comes.
D6's "whichever copy VS Code runs, the result is the same" does not hold.

### [MEDIUM] Where the CSP lives and how the page picks its mode are unstated

**Location:** `_lym/discussion/decision-record.md:226` (D9), with D2 lines 61–63 and D4 line 118

**Issue:** This merges walkthrough F2 and interactions F3.
The single asset loads `board-data.js` by a script tag.
A `<meta>` CSP with hash-only `script-src` blocks that tag at file://.
A header CSP leaves the tag requesting `/board-data.js` over http, a path no D10 route serves.
Over http, `window.BOARD` is then undefined, and D4's rule shows the `lym board rerender` line.

### [MEDIUM] An older binary replays every newer sync commit as a plain cherry-pick

**Location:** `_lym/discussion/decision-record.md:79` (D3 bound)

**Issue:** D1's projection carries the render time, so every sync commit by an upgraded machine touches `board-data.js`.
D3 states that "a commit touching an unclaimed path is replayed as a plain cherry-pick", and an older binary claims no such path.
For as long as one machine lags, every commit replays without the by-entry merge, which D3's bound does not state.
D3's bound covers only the downgrade's removal commit.

### [MEDIUM] Releasing a running task from the page has no stated consequence

**Location:** `_lym/discussion/decision-record.md:264` (D10 bound)

**Issue:** The bound states what release lets through: a running task.
It does not state what follows.
`applyDraftRule` leaves a nil status startable, so one click hands a task to its parent's dobby while its run still holds its pair.
The rubric's attack-surface test asks for what the edge lets through and what bounds it, and this edge has no bound.

### [MEDIUM] Tool facts are stated without a pin

**Location:** `_lym/discussion/decision-record.md:61` (D2), lines 106–110 (D4), 140 (D5), 156 and 178–188 (D6), 226 (D9)

**Issue:** Each of these Decisions rests on how a browser, xdg-open or VS Code behaves, and none names the test, recorded output or documentation that pins it.
Every later round would otherwise repeat the check.
D6's multi-root claim matters most, because the one-key goal depends on runTask running one `board open` without a picker.
D9's hash-only CSP silently forbids inline event handlers and style attributes, and the plan writer needs that constraint.
D1's JSON escaping is pinned by its criterion and needs nothing.

### [MEDIUM] The Open risks line is the parent's call

**Location:** `_lym/discussion/decision-record.md:392` (Open risks)

**Issue:** The entry's Delivers line names the refusal, and the operator's "Wanted next" section makes it moot.
Dropping a named deliverable, and asking whether two named render outputs still serve a purpose, go to whether the entry still says what the task is for.
No Added after Discussion entry answers either question, and `lym board get board-viewer-build` shows no parent decision note.
The question goes to the run's parent.

### [MEDIUM] An orch's `open` can block on a foreground browser

**Location:** `_lym/discussion/decision-record.md:140` (D5 bound), with D12 line 307

**Issue:** D12 lets an orch session run both verbs.
On a machine where xdg-open detects no desktop, D5 waits on a foreground browser with no cap.
The bound's only stated consequence is VS Code's running-task prompt.
PATTERN-daemon-wakeups binds a wait an agent blocks on to the keep-warm cap.

### [MEDIUM] Criteria miss a trust check and several stated refusals

**Location:** `_lym/discussion/decision-record.md:429` (Acceptance criteria)

**Issue:** This merges interactions F5, F6 and F7.
The file route's `t` token is the one trust check with no criterion.
D6's two refusals that write nothing have no criterion either.
D10's page behaviour sits outside D4's `Review:` criterion.

### [LOW] The revision a write returns is not pinned to what the next check computes

**Location:** `_lym/discussion/decision-record.md:254` (D10), D8 lines 195–204

**Issue:** The existing-paths lens found the `Task` encoding stable across a load, so the risk is small.
Still, the record names no source for the answer's revision, and no criterion covers two edits in a row.

### [LOW] The render's undiscussed copy versus the revision and the file names

**Location:** `_lym/discussion/decision-record.md:37` (D1)

**Issue:** `internal/boardengine/render.go:44` and `:73-85` clear `Discussed` in a copy for an entry whose decision file is missing.
The state is rare, so the existing-paths lens's MEDIUM is lowered to LOW.
The record should still say which entry the revision is taken from, and which decision-file names the projection lists.

### [LOW] The server's `runproc` role is unnamed

**Location:** `_lym/discussion/decision-record.md:292` (D11), D13

**Issue:** This merges existing-paths F2 and bound-breaker F6.
`internal/runproc/end.go:96-100` spares the hub-wide roles.
The takeover claim holds only for a role outside that list.

### [LOW] A create refused for a raddle-store slug has no entry to carry

**Location:** `_lym/discussion/decision-record.md:205` (D8)

**Issue:** `internal/boardengine/board.go:120` and `:172` judge the raddle store in `checkNewSlugs`.

### [LOW] The support log contradicts D9's and D12's bounds

**Location:** `_lym/discussion/support-log.md:104` (Q7), and the Rejected alternatives bullet at line 89

**Issue:** Turn 4 says D12's rationale no longer claims the refusal closes the path.
Two other places in the log still claim it.

### [LOW] Notes restate D6 and D8

**Location:** `_lym/discussion/decision-record.md:447` (Notes for the plan writer)

**Issue:** The plan writer reads both, so the restatement adds nothing.
Only the file location of D8's check is new, and the fix keeps it.

### [NIT] D3's last bound sentence is moot

**Location:** `_lym/discussion/decision-record.md:90` (D3)

**Issue:** The render time already changes `board-data.js` on every write.

## Rejected

- No lens finding was rejected outright.
  The existing-paths lens's D1/D8 copy finding was kept, with its severity lowered to LOW as its prose states.
  The interactions lens's drain finding was merged into F3.

## Settled focus questions with no finding

- Focus 1: `GET /` carries no board data and is bounded by the Host check.
  The file route is bounded to names the projection lists, and the launcher by its mode, apart from F2.
  The idle exit is bounded at 30 minutes.
  Only the drain window (F3) and the Windows token (F2) lack a stated bound.
- Focus 3: D9's "fixed byte cap", D5's bounded start wait, D11's "few seconds" lock wait and its page retry time are plan-level values, not facts the design rests on.
  No two of them must be ordered against each other.
  `fswatch`'s rename handling is internal code, as its `Watch` doc states, not a tool fact.
- Focus 4: no Decision edits this task's own board entry, files a task or promotes a note.
  The page's create and promote are the feature, run by the operator.
- Focus 5: the Added entries that name D7, F7 and F8 are answered by the parent's two Added entries and explained by the Notes line on D7.
  No remaining text depends on a missing decision, and the one Open risks line is F9.
- Focus 6: D3's and D6's long bounds are kept.
  Each states what a new routing edge, the replay claims and the user-global keys, lets through, as the spec requires.
  Only F17 and F18 are relocation or cut findings.

## Note on method

Step 1 required reading without judging until the forks were spawned.
I formed several of the handler findings above while reading in step 1, before the spawn, so the forks inherited that context.
