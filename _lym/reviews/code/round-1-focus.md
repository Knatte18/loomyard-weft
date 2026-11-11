---
round: 1
focus:
  - "internal/creelengine/verbs.go and server.go: does every post path (strand, parent, broadcast or multi-recipient, reply) compute and return receiver_live, and can a liveness lookup failure or race fail the post or report a wrong value?"
  - "internal/creelcli/cli.go: does the warning line name the strand and how to start it (`lym beam start` for an orch), and is it emitted only when the recipient is not live, without breaking the existing JSON output shape?"
  - "internal/creelengine/protocol.go: is the new reply field backward compatible with existing clients and the daemon protocol, and does the message still queue when the recipient is not live?"
  - "Test economy: in internal/creelengine/verbs_test.go, server_test.go and internal/creelcli/cli_test.go, do the new tests cover the not-live and live cases without duplicating existing coverage, and is any added behavior left uncovered?"
  - "Docs: do creelcli/doc.go and creelengine/doc.go, and any help text or spec the repository's rules name for a CLI output change, describe receiver_live and the warning accurately?"
---

Scope is issue #96: `creel post` to a non-live strand still queues, but reports `receiver_live` and warns.
