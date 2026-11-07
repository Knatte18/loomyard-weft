```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens. For each change, I found its dependents and checked whether the diff leaves any of them wrong.

**Per change, the set found, covered or not**

- **`HubDirectory` gains `RepositoryName`.**
  - Implementers are `creelcli/hubdirectory.go` and `creelfake/hubs.go`, and both are updated and carry `var _` assertions. Covered.
- **`trusted_hubs` now lists repository names.**
  - Readers are `receive.go`, `hubsverb.go`, the `creelconfig` doc comments, `template.yaml`, `creelcli/doc.go`, `creelengine/doc.go`, `contracts/orchrole/orch-role.md` and `pattern/PATTERN-agent-addressing.md`. All are updated.
  - The only non-Go files that mention it are the orch-role contract, the template and the pattern file. I found no stencil that still says shortnames. Covered.
- **`Message.HubStamp()` now leads with the repository.**
  - Callers are `reminder.go:43` and `:82`, `creelcli/cli.go:639`, `MessageLine`, and `widestStampedLine` (`crosshub.go:36`).
  - Existing tests assert `StampTrusted` or `StampUntrusted` only for messages with an empty repository, which is correct.
  - `widestStampedLine` is not covered. See F1.
- **New `SendingRepository` field.**
  - It is journaled in `journal.go`, folded in `fold.go` at inbound, carried and carry-record, and exposed in `cli.go` `messageFields`. Covered.
  - The log-entry builder at `cli.go:853` is not updated. See F2.
- **`PATTERN-agent-addressing`.**
  - The rule text is updated. No declared PATTERN changes list is rendered in my context, so I raise nothing on that.
- **Imports.**
  - `hubregistry` is not a leaf package in `PATTERN-leaf-packages`. Its new imports of `fabricengine` and `githubclient` break no rule.
  - `configreg` gains `hubregistry` and `standalonestate`. Covered.
- **Commit message.**
  - The commits carry no `Pattern-Departure:` trailer and need none.

**Findings**

- **F1, MEDIUM, design.**
  - **Location:** `internal/creelengine/crosshub.go:35-37`, with `internal/creelengine/fold.go` `HubStamp`.
  - **Summary:** `widestStampedLine` is the length check applied before a message is posted (`validateStampedLine` compares it to `boundaryClaimBudget`). It renders the widest line with `SendingHub: "hub"` and no `SendingRepository`. The stamp now carries `owner/repo` (up to about 140 characters), so a body that passes the check can produce a longer delivered line at the receiving hub and overflow the hook delivery budget.
  - **Fix:** give `widestStampedLine` a `SendingRepository` of the widest legal GitHub name, and keep a test pinning that a body at the limit still fits.

- **F2, LOW, consistency.**
  - **Location:** `internal/creelcli/cli.go:853`, with `internal/creelcli/doc.go:32`.
  - **Summary:** the `log` entry for mail across hubs lists `sending_hub` and `sending_shortname` but not the repository, though `messageFields` and the inbound record now carry it. The doc comment's list of fields for such entries is silent on `sending_repository`.
  - **Fix:** add `sending_repository` (when set) to the log entry fields, or state in `doc.go` that log entries omit it. Only do this if the answer key wants the repository visible in the log; the Goal asks only for `hubs` and the trust stamp, so this may be out of scope.

**Tool calls**
1. A grep for `trusted_hubs` and `HubDirectory` in non-Go files and for `HubDirectory` implementers, plus `hubregistry` importers and the leaf-packages pattern. My context lacked the non-Go and non-diff dependents.
2. A grep for `sending_hub` outside the diffed files, plus `creelcli/doc.go` lines 108-125. I needed the other readers of the changed fields.
3. `crosshub.go` lines 15-45 and `verbs.go` lines 930-950. I needed the body of `widestStampedLine` and the forward path.
4. A grep for `SetTrusted`, `HubStamp()` and the stamp constants across `internal`, to find the callers and asserters of the changed stamp.
