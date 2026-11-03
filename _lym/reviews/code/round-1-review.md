---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/hibernateengine/verbs.go:93
    summary: Submit replaces a finished request without lifting or refusing over the pause entry that request kept, so cancel can no longer lift it
    fix: In Submit's updateRequest callback, when found and cur is terminal and cur.Owned is not nil, read deps.TenderStatus and refuse with a way-forward message naming "lym hibernate cancel" or "lym tender resume" when that owned entry (matched by Scope, Caller and PausedAt) is still live; replace the request as before when it is not live. Update the Submit doc comment and the engine doc.go Submit paragraph, drop the matching Known limits sentence from the change description, and add a TestSubmit row for a done request with a live owned entry, refused with the file unchanged, and one with the entry already lifted, replaced.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/hibernateengine/watch_test.go:570
    summary: No row pins that a failure before the hibernate still lifts the entry for a request that does not resume on wake, nor the advance-failure call site of failAtWake with the switch off
    fix: Add rows to TestWatch that seed the request with ResumeOnWake false through watchKeeping. One row makes the pause or the confirmation fail before the hibernate and asserts the hub entry is lifted and the notice says its pause was lifted. One row makes the write of the resuming phase fail and asserts the entry stays live and the notice carries keptPauseOutcome.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/hibernateengine/watch.go:148
    summary: On the keep path, an owned entry whose record write failed is kept live but the request is written done with Owned nil, so cancel cannot lift it
    fix: In Watch, when r.ResumeOnWake is false and r.Owned is nil and kept is not nil, call recordOwned(deps, *kept) before finish and carry the written request forward, so the done request records the entry it kept; on a recordOwned error keep the entry live and return the error joined with the notice error.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/hibernateengine/watch.go:219
    summary: The keep path's notice for a request that owns no entry has no row, and neither has a keep-path failure with no entry to name
    fix: Add a TestWatch row that seeds an off request whose pause entry was never recorded as owned, runs Watch to the wake, and asserts the request ends done with the notice naming that it owned no pause, plus one off-path failure row with no owned or kept entry asserting "it owned no pause to lift."
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/hibernatecli/cli.go:116
    summary: A hibernate.yaml that fails to load refuses the request, and no test pins that refusal
    fix: Add a row to TestChooseResumeOnWake with an unparsable hub file and the flag absent, asserting a non-nil error and that the flag given still wins without reading the file.
    origin: lens:code-tests
---

### [MEDIUM] Submit drops the Owned reference of a kept pause

**Location:** internal/hibernateengine/verbs.go:93-100
**Issue:** With the switch off, a request that ends done or failed leaves its hub entry live and recorded as owned, and the Goal names `lym hibernate cancel` as the way to lift it. `Submit` replaces a terminal request without a lift or a refusal, so a second `lym hibernate request` overwrites the file and loses the reference. The entry then stays live beside the new request's own pause, and only `lym tender resume` lifts it. The change description lists this as a known limit, but the diff turns a kept pause from a rare failure leftover into the default end state, so the gap now meets the ordinary flow.

### [MEDIUM] Failure paths of the switch's scope are not pinned

**Location:** internal/hibernateengine/watch_test.go:570
**Issue:** The off-path rows `watch_test.go:570` and `watch_test.go:589` cover a successful wake and a failed status read inside the reconcile. Every other `TestWatch` row runs with `ResumeOnWake` true through `ownRequest`. A change that made `fail` follow the request's switch before the hibernate, or one that reverted the `advance` failure to `fail`, would pass the whole suite. The first is the boundary the description claims, that a failure before the hibernate lifts the entry whatever the switch says.

### [LOW] A kept entry whose record write failed ends unrecorded

**Location:** internal/hibernateengine/watch.go:148-164
**Issue:** `kept` holds the entry when `recordOwned` failed, and `target.Owned = kept` stands in for it only for the reconcile. On the keep path nothing writes it back, so `finish` writes done with `Owned` nil while the notice says the pause was kept. `lym hibernate cancel` then lifts nothing, and only `lym tender resume` does.

### [LOW] Keep-path notice branch without an owned entry is untested

**Location:** internal/hibernateengine/watch.go:219-224
**Issue:** `keptOutcome` returns the owned-no-pause sentence when `Owned` is nil, a branch the diff adds. The only row that reaches it is for the lifting path, so a regression to the kept sentence would pass.

### [LOW] Config load refusal of hibernate request is untested

**Location:** internal/hibernatecli/cli.go:116-123
**Issue:** `chooseResumeOnWake` returns the `LoadConfig` error, and `request` refuses on it. The diff adds this refusal, and no row feeds it an unparsable `hibernate.yaml`.

## Rejected

- The off-path notice claims a kept pause without reading whether the entry is still live (code-correctness, finding 3): the window between the reconcile and the notice is short, the on path's report is the only live reading in that code, and the Goal asks only that the notice say the hub stays paused.
- A request file with no `resume_on_wake` key reads as off, with no decoding test (code-tests, behaviour 17): the zero value of a JSON bool is language behaviour, and the description names the compatibility, so a test would pin the decoder and not the diff.
- `lym hibernate status` shows `resume_on_wake`, with no assertion (code-tests, behaviour 16): the key follows from the struct's JSON tag, and `TestSubmit` pins the field's value.
- Cancel after a kept pause for the failed phase (code-tests, behaviour 11): the row at `verbs_test.go:299` loops over done, failed and cancelled.
- `wantLocalKeys` lacks a `hibernate` entry (code-tests, behaviour 13): the key is cross-pair, so `LocalKeys` is empty and equals the absent map entry.
- A dead daemon's non-terminal request in the resuming phase has its owned entry lifted by a new `Submit` whatever its switch (handler, focus bullet 2): the new request pauses the hub again, `Submit` lifts only for a daemon that cannot finish, and the Goal's keep rule covers a wake the daemon finished.
- The `configcli.go` Long-page edit and the `PATTERN-config-layers.md` degrading-list edit are licensed by the commits that carry them: the byte cap fix, and the new degrading caller.
- The edits to `contracts/orchrole/orch-role.md` and `docs/overview.md` match the behaviour and drop no existing instruction.
