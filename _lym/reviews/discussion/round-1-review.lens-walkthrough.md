```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism on one concrete case, end to end.

## List

- **D1.** Case: one note, run through `Render`, then `board-data.js`, then `JSON.parse` in the page. Purpose met. `Outputs` carries only `Types`; area labels are those outside `Types`, so the plan can derive them.
- **D2.** Case: `board.html` opened at `file://`, and the same bytes served at `GET /`. Purpose at risk: see F2.
- **D3.** Case: an upsert, then `RenderToDisk`, then the manifest, then `commitDirty`, then a replay through `Claims`. Purpose met. The owner-set path renders through `Render`/`RenderToDisk` (`ownerset.go:331,336`).
- **D4.** Case: a body link `[x](javascript:…)`, then escape, trim, scheme check, plain text. Purpose met.
- **D5.** Case: a second key press while the first press's server is still starting. Purpose missed: see F1.
- **D6.** Case: `alt+shift+0` in the hub workspace window runs `lym beam start root` from the prime's anchor. Purpose met: `beam start` takes "root" as the prime (`internal/beamcli/cli.go:475`).
- **D8.** Case: page edit with the revision from `GET /api/board`. The critical section loads, migrates (`board.go:116-117`), hashes, compares, then runs the mutation. Purpose met.
- **D9.** Case: launcher at `file://`, `location.replace` to `127.0.0.1:&lt;port&gt;/#t=…`, the page reads the fragment, then a Bearer `/api/board` call. Purpose met, but the CSP's location is ambiguous: see F2.
- **D10.** Case: release a draft task with `status` `null` through `SetStatus` (`store.go:682`). `applyDraftRule` re-drafts only new tasks. Purpose met.
- **D11.** Case: idle page. The long poll, under a minute, repeats, so the 30-minute idle exit never fires while a page is open. Purpose met.
- **D12.** Case: `open` from an orch session. The server inherits the orch's env, D12 passes the board an empty strand name, and the writes are the operator's. Purpose met.
- **D13.** Case: a write path from `boardserve` to `boardengine`'s exported methods. Purpose met.

## Findings

**F1 — MEDIUM, design, `_lym/discussion/decision-record.md` D5 (lines 128–129) against D11 (lines 276–280).**
D5's live-server read trusts a state file that may still be the previous server's, so a second press during startup opens a dead page.
- D11 orders the server's start as: take the lock, build the board, register, bind, and only then write the state.
- D5's live path reads "lock held" as proof that the recorded port is the server's, and hands the launcher over without checking.
- Counterexample: the first `board open` spawns server S. S takes the lock and is still building its board or binding when the operator presses the key again.
- The second `open` finds the lock held and reads the state file left over from the last dead server: old port, old token. It writes a launcher for that address.
- The page fails, retries for D11's bounded time, then shows the "lym board open" line, although a server is coming up. If another process now holds the old port, the page loads that process's content.
- Fix: the live path also waits, bounded, until the state answers a request carrying its token, as the start path does. Or the server writes a fresh state before it takes the lock. Add a criterion for the press made during startup.

**F2 — MEDIUM, design, `_lym/discussion/decision-record.md` D9 (line 226) against D2 (lines 61, 63) and the D2 criterion (line 407).**
The record never says whether the page's CSP is a response header or a `&lt;meta&gt;` in the embedded asset, and either reading breaks something.
- D2 has one embedded asset serve both modes, loading `&lt;script src="board-data.js"&gt;` at `file://`.
- D9 says the page "carries a `Content-Security-Policy` that allows only its own inline script and style by their hashes."
- If the CSP is a `&lt;meta&gt;` in the asset: it applies at `file://` too, and a hash-only `script-src` blocks the external `board-data.js`. That breaks the read-only `file://` mode D2 promises.
- If it is a header: over `http` the same `&lt;script src="board-data.js"&gt;` asks for `/board-data.js`, which no D10 route serves and the CSP blocks. The page must then pick its mode by protocol, never by `window.BOARD` being undefined, or it shows D4's `lym board rerender` line over `http`.
- Fix: state that the CSP is a response header only, never a `&lt;meta&gt;` in `board.html`. State that the page picks its mode by `location.protocol`.

## Tool calls
1. Grepped the `Outputs` struct, `newBoardView`, the `Render`/`RenderToDisk` calls in `ownerset.go` and `beam start`'s `Use`. My context had none of D1's render inputs, D3's owner-set path or D6's verb.
2. Read `internal/beamcli/cli.go` lines 465–500, to confirm that `lym beam start root` names the prime, which D6's workspace task depends on.
