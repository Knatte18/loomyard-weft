---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/boardserve/run.go:424
    summary: "The watch loop wakes and re-reads and hashes board.json on every fswatch event with no one-second floor, which D11 and PATTERN-daemon-wakeups require."
    fix: "In watchBoard (internal/boardserve/run.go), after a wake caused by an event, skip further reads until a one-second floor (a named constant, waited through opts.After so a test can inject it) has passed since the previous read: drain or ignore events inside that window, then read board.json once. fswatch coalesces nothing (internal/fswatch/doc.go), so the caller owns the floor, as dobby's wakeFloor does in internal/dobbyengine/loop.go. Add a TestRun row that sends a burst of events and asserts one read per floor, and one that asserts the After durations double from one second to eight and reset to one on a change."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/board.html:966
    summary: "With no board.json the server answers revision empty, the page polls with no after parameter, the server answers at once, and the page re-requests and re-renders in a tight loop that also keeps resetting the idle clock."
    fix: "Make the poll distinguish an absent revision from no after. In board.html, send after= whenever the previous answer carried a revision string, even an empty one, and send no after only on the first request. In internal/boardserve/server.go, have serveBoard pass whether the after key is present (r.URL.Query().Has(\"after\")) to waitForChange, and let waitForChange return at once only when after is absent or differs from the current revision. Add a TestServer row: an empty board.json revision with after= present waits for the bound and answers unchanged."
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: .gitattributes:7
    summary: "board.html is a go:embed target but is not pinned to LF, which the repository's .gitattributes comment requires; a CRLF checkout changes the embedded bytes, and the browser's newline normalization then makes the page's CSP script and style hashes miss, so the page's script is blocked."
    fix: "Add the line `internal/boardengine/board.html text eol=lf` beside the other go:embed entries in .gitattributes, as is done for internal/boardengine/template.yaml."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/boardcli/serveopen.go#serveCommand
    summary: "D12 and D5 behaviors in the verb wiring have no covering test: an orch strand name proceeds past the refusal, the server's board is built with an empty strand name, productionOpenDeps sets OpenerCap from the keep-warm resolver for a strand-named caller, and the production Register closure registers under HubOwner with RoleBoardServer."
    fix: "In internal/boardcli, add rows to TestServeOpenRefusals (or a sibling table test) that run refuseBoardVerb with an orch strand name and with none and expect a pass, and assert that productionOpenDeps(location) with a named strand name returns a non-zero OpenerCap and with none returns zero. Make the server board's strand-name clearing and the Register closure reachable from a test (for example a small function that builds the Options from the config and location) and assert StrandName is empty and Register records RoleBoardServer under runproc.HubOwner in a temp run-process directory."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/boardserve/server.go:140
    summary: "ServeHTTP stamps the idle clock for every /api path before the token check, so an unauthenticated or wrong-token request keeps the server alive past the 30-minute idle bound."
    fix: "In internal/boardserve/server.go, move the lastAPI stamp so it runs only after bearerAuthorized succeeds on /api/board and /api/write; keep the BoardReadIsTheProjection assertion and add a TransportRefusalsChangeNothing check that a refused request leaves lastAPIRequest unchanged."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/boardserve/write.go#checkRequest
    summary: "A create request that carries a non-empty revision runs Expecting with that revision, so a create of an existing slug becomes an edit and skips D8's create-of-existing-slug refusal."
    fix: "In checkRequest (internal/boardserve/write.go), refuse a non-empty revision when the op is create, with a message naming the way forward (send a create with no revision). Add a RefusedWritesChangeNothing row for it."
  - id: F7
    severity: LOW
    class: design
    origin: handler
    location: internal/boardengine/expect.go#checkExpected
    summary: "A create whose slug names a project passes the existence check, finds no entry in the projection, and returns a StaleEntryError with a nil Fresh entry; the server answers 409 with entry null and the page treats that as a removed entry and shows its removed panel."
    fix: "In checkExpected, when revision is empty and the slug exists but is not a task or note (a project), return a plain error naming the slug as taken by a project and the way forward (pick another slug), not a StaleEntryError. Add a row to TestExpectingRevision for a create of a project's slug."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/render.go#RenderToDisk
    summary: "D3's acceptance that a board write's sync commit holds board-data.js and board.html beside README.md, and that the owner-set write path commits every name Render returns, has no covering test."
    fix: "Add a test in internal/boardengine that makes a board write with git enabled in a hermetic Bolt fixture and asserts the resulting sync commit's file list holds README.md, board-data.js and board.html, and one that does the same through the owner-set write path."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/hubworkspace.go#regenerateHubWorkspace
    summary: "The branch that logs a warning and writes a beam's workspace file with no hook members when the beam's anchor cannot be read has no test row."
    fix: "Add a row to TestRegenerateHubWorkspace in internal/fabricengine/hubworkspace_integration_test.go that installs a hook, makes one beam's anchor unreadable, and asserts that beam's file carries no hook member while the hub's file and the other beams' files carry theirs."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/boardengine/view_test.go#TestRenderBoardData
    summary: "EntriesCarryTheirStoredValues computes the expected revision by calling EntryRevision, the function under test, so D8's definition (the hex SHA-256 of the entry's JSON) is pinned by no independent value."
    fix: "In the EntriesCarryTheirStoredValues row, compare each entry's revision with hex(sha256(json.Marshal(stored task))) computed in the test, or with one literal digest for a fixed entry."
  - id: F11
    severity: LOW
    class: consistency
    origin: handler
    location: internal/boardengine/viewer_test.go#TestViewerPage
    summary: "The redundancy report lists TestViewerPage as contained, but it alone asserts that every render writes board.html byte for byte, that the page has a script tag for board-data.js, and that it names no http or https URL; its covering tests assert none of these."
    fix: "Add the comment line `//testtiming:keep pins that every render writes the embedded page bytes and that the page loads its data by script tag with no outside URL` directly above `func TestViewerPage`."
  - id: F12
    severity: LOW
    class: consistency
    origin: handler
    location: internal/vscode/keybindings_test.go#TestKeybindingsBlock
    summary: "The redundancy report lists TestKeybindingsBlock as contained, but it alone asserts the block's literal entry text; TestMergeKeybindings builds its expectations from keybindingsBlock itself."
    fix: "Add the comment line `//testtiming:keep pins the block's literal entries, which the merge test builds from the block itself` directly above `func TestKeybindingsBlock`."
---

### [BLOCKING] The watch loop has no one-second floor on event wakes

**Location:** internal/boardserve/run.go:424
**Issue:** `PATTERN-daemon-wakeups` and D11 require an unbounded wait to wake at most once a second.
`watchBoard` wakes on every `fswatch` event, which the package never coalesces or debounces, and then reads and hashes `board.json`.
One board write yields several filesystem events, so a burst produces a burst of wakes.
The pattern entry the diff adds to `pattern/PATTERN-daemon-wakeups.md` claims the loop wakes at most once a second only through its timer, not through events.

### [MEDIUM] The page busy-loops when board.json is absent

**Location:** internal/boardengine/board.html:966
**Issue:** `BoardRevision` returns an empty string for an absent `board.json`.
The page's `poll` then sends no `after` parameter, and `waitForChange` treats an empty `after` as "answer at once".
The reply is not `unchanged`, so the page re-renders and re-requests immediately, forever, and every request resets the idle clock.
A hub with no board entries yet reaches this state.

### [MEDIUM] board.html is not pinned to LF

**Location:** .gitattributes:7
**Issue:** The comment in `.gitattributes` says every `go:embed` target must be pinned to LF so its bytes do not depend on a machine's `core.autocrlf`.
`board.html` is a `go:embed` target and is not listed.
On a CRLF checkout, `contentSecurityPolicy` hashes the CRLF bytes, but a browser normalizes newlines before it hashes an inline element, so the hashes miss and the page's inline script and style are blocked.
The rendered `board.html` in the Bolt would also differ between machines.
The repository supports Windows, where `autocrlf` is common.

### [MEDIUM] Verb wiring behaviors have no covering test

**Location:** internal/boardcli/serveopen.go#serveCommand
**Issue:** D12's acceptance says an orch strand name proceeds past the refusal and the server passes the board an empty strand name; D5 says a strand-named caller's opener wait takes the keep-warm cap.
`TestServeOpenRefusals` runs only a run agent and an empty strand name under `--board-path`, `TestRunOpen` injects `OpenerCap` directly, and `TestBoardServeScenario` runs with an empty strand name, so a removed `boardConfig.StrandName = ""`, a broken `KeepWarm` mapping or a wrong `Register` role would all pass.

### [LOW] An unauthenticated /api request resets the idle clock

**Location:** internal/boardserve/server.go:140
**Issue:** `lastAPI` is stamped when the path starts with `/api/`, before the token is checked.
Any local process, with or without the token, keeps the server alive beyond D11's 30 minutes.

### [LOW] A create request may carry a revision

**Location:** internal/boardserve/write.go#checkRequest
**Issue:** `checkRequest` requires a revision for every op but create and does not forbid one on create.
A create with a non-empty revision runs `Expecting(slug, revision)`, so a create of an existing slug is treated as an edit instead of refused as D8 states.

### [LOW] A create of a project's slug answers like a removed entry

**Location:** internal/boardengine/expect.go#checkExpected
**Issue:** With an empty revision and a slug held by a project, the slug is found in the loaded tasks, the projection lists no entry for it, and the refusal carries `Fresh == nil`.
The server answers 409 with `"entry":null`, which the page reads as "removed" and shows in its removed panel, though the entry exists.

### [LOW] D3's sync and owner-set commit contents are untested

**Location:** internal/boardengine/render.go#RenderToDisk
**Issue:** The acceptance criterion says a board write's sync commit holds both new files beside `README.md`.
Only the manifest and the replay claims are tested.

### [LOW] The unreadable-beam-anchor branch has no test

**Location:** internal/fabricengine/hubworkspace.go#regenerateHubWorkspace
**Issue:** The branch that warns and writes a beam's file with no hook members has no row.

### [LOW] The revision test computes its expectation with the code under test

**Location:** internal/boardengine/view_test.go#TestRenderBoardData
**Issue:** `EntriesCarryTheirStoredValues` expects `EntryRevision(stored[entry.Slug])`, so D8's hash definition is pinned by no independent value.

### [LOW] TestViewerPage pins what its covering tests do not see

**Location:** internal/boardengine/viewer_test.go#TestViewerPage
**Issue:** The redundancy report lists it as contained, covered by `TestRenderBoardData` and `TestFileRaddle`; neither asserts `board.html`'s bytes, its script tag or the absence of an outside URL, so it is kept and marked.

### [LOW] TestKeybindingsBlock pins what its covering test does not see

**Location:** internal/vscode/keybindings_test.go#TestKeybindingsBlock
**Issue:** The redundancy report lists it as contained, covered by `TestMergeKeybindings`, which builds its expected text from `keybindingsBlock`; only this test asserts the literal entries, so it is kept and marked.

## Rejected

- Lens code-correctness F4 (the state file is briefly group-readable): `state.WriteJSON` writes through `fsx.AtomicWriteBytes`, whose `os.CreateTemp` file is created at mode 0600 and renamed into place, so the file is owner-only from creation on Linux; the launcher is created at 0600 by `os.WriteFile`.
- Lens code-tests finding on `PageHoldsEachRoute`: `board.html` is an executable asset with no JavaScript runtime in the suite, and the assertion pins the contract between the page's routes and the server's, not static prose of a doc.
- Lens code-affected-set finding on the first sentence of the `render.go` header: the header's later lines name `board-data.js` and `board.html`, so a reader is not misled.
- Lens code-scope: no finding; I found no change without a decision and no decision without a change.
