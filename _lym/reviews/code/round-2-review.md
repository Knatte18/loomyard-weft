---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:38
    summary: The change description says an unreadable machine record drops the trust entry and describes a hub test row the code no longer has.
    fix: In the "Failed resolutions are dropped, not kept" design decision, replace "Whatever the cause (unknown shortname, shared shortname, no origin, an unreadable machine record), the rewrite drops that entry" with text saying that only a shortname naming no hub or several (hubregistry.ErrUnresolved) and a hub with no valid repository (hubregistry.ErrNoRepository) drop the entry, while any other resolve error, such as an unreadable per-user state directory or a corrupt record, ends the rewrite with that error so no marker is written and the next walk retries. In the Tests section, replace "an untrusted hub with a local-path origin" in the `creelcli` hubs bullet with the cases the row pins now - a listed shortname, a hub with no origin and a hub whose origin names a forged repository, each untrusted with its reason - and add the `sending_repository` present and absent assertions of the `read --envelope` row.
---

### [LOW] The change description contradicts the resolve-error handling the code ended up with

**Location:** `_lym/landing/summary.md:38`

**Issue:** The "Failed resolutions are dropped, not kept" bullet says the rewrite drops an entry "whatever the cause", naming "an unreadable machine record".
`repositoryOfShortname` in `internal/configreg/configreg.go` does the opposite for that cause: `standalonestate.UserStateDir` and `hubregistry.Lookup` errors that are neither `ErrUnresolved` nor `ErrNoRepository` pass through `markUnresolvable` unchanged.
`RewriteShortnamesToRepositories` in `internal/creelconfig/reinterpret.go` returns them, and `applyReinterpretations` in `internal/hubreconcile/reinterpret.go` returns before it writes the marker, so no entry is dropped and the reinterpretation is not recorded.
`TestMarkUnresolvable` and the `AResolveFailureThatIsNoUnresolvedShortnameIsReturnedAndDropsNoEntry` subtest pin this.
The Tests section also lists "an untrusted hub with a local-path origin" for the `creelcli` hubs row, which now covers a listed shortname, a hub with no origin and a forged repository name.
The summary is the writer's claim, and the diff does not bear these two claims out.

## Focus verdicts

- **`widestStampedLine` and `validateStampedLine`** (`internal/creelengine/crosshub.go`): not a defect.
  The widest line is the untrusted form, led by a 39-character owner and a 100-character repository, and `MessageLine` renders the stamp from `HubStamp`.
  `RepositoryName` caps both segments at the same lengths, so a real line is never wider, and the receiving hub's `refuseDeliveredNames` runs the same check.
  `TestStore_Mail` pins a body at the budget with the longest repository leading the stamp (`atTheLimit`).
- **`applyInbound`, `applyCarried`, `carryRecords`** (`internal/creelengine/fold.go`, `journal.go`): not a defect.
  Each fold copies `SendingRepository`.
  The `TestJournal_Fold` inbound row asserts it and the stamp, and `TestJournal_Retention` asserts it both present and absent after the carry and after a reload.
  `record.SendingRepository` is the journal field in `journal.go`.
- **`messageFields`** (`internal/creelcli/cli.go`): not a defect.
  The field is emitted when set and omitted when empty, and the `read --envelope` row asserts both.
- **configreg registration, `repositoryOfShortname`, `LookupRepository`**: not a defect.
  `TestModules_CreelRegistersTheTrustedHubsReinterpretation` fails if the registration is dropped.
  `TestLookupRepository` covers the recorded, unrecorded, ambiguous, no-origin and corrupt-record cases.
- **Resolve errors in `RewriteShortnamesToRepositories`**: not a defect.
  Only `ErrShortnameUnresolvable` drops an entry, which `markUnresolvable` assigns to `ErrUnresolved` and `ErrNoRepository`.
  Any other error is returned, and `applyReinterpretations` writes no marker on an error, so a transient failure such as an unreadable state directory leaves the reinterpretation pending.
- **`RepositoryName` characters** (`internal/hubregistry/repository.go`): not a defect.
  `validName` admits only ASCII letters, digits, hyphen, underscore and dot, within the length limits, and refuses `.` and `..`.
  A forged name such as `Owner/(other hub, trusted orch)` or `owner/...` is refused, and `TestRepositoryName` pins both.

## Rejected

- `RepositoryName` wraps every `HubOriginURL` failure, including an unreadable prime config, as `ErrNoRepository`, so such a hub's entry is dropped and the marker written.
  The decision record says a shortname that resolves to none is dropped with a line naming it, and the direction (losing trust) is the safe one, so it is no finding.
