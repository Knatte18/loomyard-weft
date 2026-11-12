---
round: 4
focus:
  - "Hour of a machine-outbound message (D3, D7 of decision-record.md): the scheduler's start scan and post wake skip a machine-outbound message and the take verb is its only reader, while `outboundWorker.run` in `internal/creelengine/outbound.go` is the code that returns a message at its hour. Settle which creel component returns a pending machine-outbound message at its hour when the relay is not running, and whether the record states it and a criterion checks it."
  - "Slack constraint on archived channels (D4 of decision-record.md): the bridge-archived notice tells the operator how to free a held channel name. Settle whether that way forward is possible under Slack's `conversations.rename` rules for an archived channel, and whether the D4 acceptance criterion has a path that reaches it."
  - "Journal form of the relay verbs (D3 of decision-record.md): the record fixes the event that creates a machine-outbound message. Settle whether it states how report, settle delivered and settle returned are journaled, including a repeated unchanged hold reason at the five-second poll, a settle delivered after a return, and an older binary reading the events."
  - "Bridge-channel hold and notice state (D4 bridge map fields, D8 of decision-record.md): the record promises one notice per episode and keeps the hold per channel. Settle where the per-channel notice marks live, including for an unmapped channel, across a relay restart, and whether an approval that delivers held messages moves the delivery cursor so a later revoke posts no second notice."
---

Round 3 was not converged. The focus entries restate the question and the sites they depend on.
