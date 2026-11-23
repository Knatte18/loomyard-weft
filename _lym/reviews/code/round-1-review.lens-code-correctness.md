```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran one concrete case through each function and branch the diff adds or changes and checked the result.

**Per function and branch: the case I ran, correct or not**

- **`networkFailure`** (`internal/sheddriver/driverules.go:306-316`)
  - An OK stop returns false: correct.
  - A failed stop with Transient `git-transport` or `github-api` returns true: correct.
  - A failed stop with Transient `agent-start` or `""` returns false: correct.
  - A status-built stop reads Transient from `st.Transient` (`driverules.go:155`), so a restarted driver classifies it the same way: correct.
- **`decideDrive`, wait row** (`driverules.go:289-290`)
  - RepairCount 0, 1 and 2 index `networkRestepWaits` to 15s, 30s and 1m: correct.
  - RepairCount 3 fails the `&lt;` guard, so there is no index out of range: correct.
  - A wake-stamp reset to 0 (`drive.go:799-805`) restarts the schedule at 15s: correct.
  - The changed-build hand-over, `ResumeReturned` and `ArbiterFirst` rows all come before the network rows, so their order is unchanged: correct.
- **`decideDrive`, exhausted row** (`driverules.go:291-292`)
  - RepairCount 3 or more hands back with NetworkNotice and makes no tackler call: correct.
  - Other transient classes still fall to the tackler row at `:293`: correct.
- **`settleStop` wait branch** (`drive.go:289-293`)
  - Wait 0 for every non-network decision skips the wait, so the old paths are untouched: correct.
  - Wait 15s with the seam returning nil leads to `holdPause`, then `recheck`, then a re-step record: correct.
  - When the context ends during the wait, it returns `("", true, ctx.Err())` with no settle record or hand-back. A later driver repeats the same wait index because no restep record was written. This matches `holdPause`'s existing error handling: correct.
  - A recheck after the wait that finds the run moved or running elsewhere continues or exits; the wait was spent, but nothing is left inconsistent: correct.
- **`wait`** (`drive.go:982-995`)
  - With the seam set, it delegates: correct.
  - Unwired, a timer fires after the delay, a cancelled context returns its error, and `defer timer.Stop()` releases the timer: correct.
  - Delay is always above 0 at the call site: correct.
- **`handBackStop` with NetworkNotice** (`drive.go:966-968`)
  - The relay notice is set, so `driveNotices` sends it with no question: correct.
  - A recheck failure on a network hand-back decision keeps both the cause rule and the notice: correct.
  - The notice leaves out the stop report path (finding 2).
- **`networkNotice`** (`drive.go:975-978`)
  - With the test's stop it produces the expected text: correct.
  - With a multi-line `stop.Error`, such as `"...sideband packet\nfatal: early EOF"`, the newline is embedded in the notice as is. This is cosmetic and gets no finding.
- **`networkRestepSpan`**: 15s + 30s + 1m = 1m45s: correct.
- **`transportPhrases`** (`internal/gitexec/transport.go:34-44`)
  - The issue's HTTP/2 line matches: correct.
  - 403 and 404 do not match: correct.
  - `"unexpected disconnect while reading sideband packet"`, `"send failure"` and `"early eof"` also match permanent refusals (finding 1).

**Findings**

1. **MEDIUM, design.** Location: `internal/gitexec/transport.go:36-41`.
   - Summary: the new phrases also match failures that are not transient.
     - A push GitHub refuses as too large (HTTP 413 or the pack size limit) prints `send-pack: unexpected disconnect while reading sideband packet` or `Send failure: Broken pipe`.
     - A fetch whose server-side pack-objects dies prints `fatal: early EOF`.
   - Both now classify as git transport failures. The driver then re-steps them three times over 1m45s and tells the parent "a network failure outlasted 3 re-steps", where before they went to the tackler as refusals.
   - Evidence:
     - `IsTransportFailure` matches any listed substring (`transport.go:60-65`).
     - `shedtransient.Class` maps a match to `TransientGitTransport` (`internal/shedtransient/shedtransient.go:34-35`).
     - The fabricengine push classifiers read a transport failure as an owed push rather than a refusal (`internal/fabricengine/pushclassified.go:332,536,635`).
   - Fix: tie these phrases to a transport-only context, for example require `early eof` or the sideband disconnect without an HTTP 4xx `RPC failed` line. Otherwise, add rows that pin the 413/too-large cases as false.
2. **LOW, design.** Location: `internal/sheddriver/drive.go:966-968`.
   - Summary: an unwatched network hand-back no longer names the stop report. Because the relay notice is set, `driveNotices` skips the `"Run %s stopped; stop report %s."` line it adds to every other unwatched hand-back, and `networkNotice` does not carry the report path.
   - Evidence: `internal/sheddriver/drivehandback.go:53-65` and `drive.go:976`.

**Tool calls**

- grep of `stopFromStatus` and `Relay` in sheddriver: my context lacked how a status-built stop carries Transient and Error, and how Relay is used.
- `sed` of `drivehandback.go:25-110`: I needed `driveNotices`' logic to see what a relay notice replaces.
- grep of the `driveNotices` callers and the notify path: I needed how the notices reach the parent.
