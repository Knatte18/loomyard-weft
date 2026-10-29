```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked every bound, behaviour and code fact that the fix added or rewrote against HEAD and the 2.1.312 bundle. Two claims fail.

**[MEDIUM]** The decision record's D3 paragraph and Constraints claim that unjournaled mail is handed out again at a later claim.
- Changed text in D3: "mail the daemon did not journal stays unread and reaches the model once more at a later claim, the visible duplicate `hook-read` already accepts".
- Changed text in Constraints: "so mail the daemon did not journal is handed out once more, as `hook-read` does today".
- Summary: this is false when the daemon refuses the acknowledgement because a journal write failed. Ids after the failing one stay reserved, are never released, and no later claim hands them out.
- Evidence: `Reading.Ack` (`internal/creelengine/verbs.go:471-486`) deletes each id's reservation and then returns at the first `journalReadLocked` error. The ids after that one keep their `store.reserved` entry, and `r.ids` stays set.
- Evidence: `handOut` (`internal/creelengine/server.go:470-472`) logs `Warn` and writes the error response without calling `Release`.
- Evidence: the claim query skips reserved ids (`internal/creelengine/pool.go:446`, `s.reserved[id] == ""`).
- So in the refused case, today's `hook-read` does not give a duplicate: mail is either shown again (the failing id) or stuck reserved (the later ids). The record does not ask the ring path to release on a refused acknowledgement either. D3's release rule speaks of ends "before the daemon journals the acknowledgement", which a partial journal does not clearly cover.

**[MEDIUM]** D6's rationale relies on every woken turn ending at a Stop.
- Changed text: "every woken turn ends at a Stop, so the echo shows at the latest when the woken turn ends".
- Summary: a turn that an API error ends fires `StopFailure` instead of `Stop`, so the echo shows only at a later turn's first claim, or never if the session ends first.
- Evidence: the 2.1.312 hook-event table says `StopFailure` "Fires instead of Stop when an API error (rate limit, auth failure, etc.) ended the turn".
- The first D6 Bound repeats the same timing ("at the woken turn's first tool call or its end") and has the same gap.

Every other claim the fix made holds:
- **D4 row component:** `Rp` and `kp` draw the summary, the duration and a bash task's delivery note. The body appears only for a monitor event (`fp` needs the `B3t` prefix, a closing `"` and the `ixn` tag), or when verbose and the notification carries a `taskDelivery`.
- **D4 stdout `rewakeSummary`:** it is gated by `rRe(pluginId)` against the `A5e` marketplace set.
- **Merged Stop command:** an exit other than 0 records the turn end. `recordingCommand` writes only to the events file, so `hook-read`'s output stays as the hook's stdout.
- **Notes on the client:** `CallWatching` resumes a `VerbHookWait` after a `connLostError`, and both the ack write and `awaitAckConfirmation` return that error.
- **D5:** the journal-failure `Warn` in `handOut` exists, and a refused acknowledgement does reach the client as `errors.New(resp.Error)`.
- **Notes on flags:** `flaggedFilesLocked` exists in `internal/creelengine/flags.go`.

Tool calls:
- Read `recordingCommand` in `settings.go` and grepped `reserved[` in `internal/creelengine`: context lacked whether the recording step writes to stdout, and whether claims skip reserved ids.
- Grepped the 2.1.312 strings for `StopFailure`: context lacked whether a turn can end without a Stop hook.
