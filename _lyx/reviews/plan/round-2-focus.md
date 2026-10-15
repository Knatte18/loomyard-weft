---
round: 2
focus:
  - "Card 6 (`_lyx/plan/06-follow-bindings-button.md`): `pinBindingsLocked` runs from `pinGeometryOptionsLocked` (`internal/reedengine/windowsize.go`), which boot and the attach pre-flight (`internal/reedengine/attach.go`) both call, and the card makes it issue one `set-option` for `@lyx_follow_command` on every pin. Settle whether every test in `internal/reedengine` that asserts an exact count of the pins' `set-option` calls is listed as a member glyph and named in the Intent with its new count."
  - "Card 2 (`_lyx/plan/02-attach-follow.md`) gates the attach selection on no other client being attached or `followingLocked` reading on, and `warnMismatchedClientsLocked` reports another client when `list-clients` fails. Settle whether the Intent decides the failed-listing case, whether `AttachArgv`'s doc comment in `internal/reedengine/attach.go` is rewritten to the follow rule, and whether a test row reaches the failed-listing-with-follow-on path."
  - "Card 3 (`_lyx/plan/03-follow-run-op.md`) plans `TestFollow_RealTmux` attaching more than one client. Settle whether each later attach is awaited until `list-clients` reports the expected client count, since a wait for any client returns immediately once the first is attached."
  - "Card 6, `TestBindings_StatusBarClicksSelectAndZoomByRangeType`: settle whether every state the live row sets up is one `pinBindingsLocked` can produce, and whether any row labelled as the fallback binding actually reaches it."
---

Focus targets the corrected text on cards 2, 3 and 6.
