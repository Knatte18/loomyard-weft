---
round: 2
focus:
  - "Search every non-test file outside internal/agentname for strings.Cut, strings.Split or strings.Index on agentname.MachineAddressSeparator, including internal/creelengine/crossmachine.go and machinemail.go: does any still form, parse or match the machine address instead of calling internal/agentname?"
  - "internal/slackengine/bridgein.go readChannel: with a held peer message followed by an approve and then a revoke in one read, is the held message delivered once after the approve, in order, with the delivery cursor moved past it, and does the revoke still stand?"
  - "internal/slackengine/bridgeout.go bridgeSender.send: is an accepted post of the record settled on every send, so an entry creel returned at its hour leaves the record without a restart, and are settle failures reported?"
  - "internal/creelengine/machinemail.go machineHoldWayForward: does every machine hold reason, including bridge-unsettled, name a way forward that exists?"
  - "internal/slackengine/bridgechannel.go ensureBridge: when a create answers name_taken and the listing does not show the channel, does the paged FindChannel listing run at most once per bridge-poll ceiling?"
  - "Rule constants for the machine name length limit and the machine envelope prefix, in internal/creelengine/crossmachine.go, internal/machinename and internal/slackengine/bridgemessage.go: is each rule defined once, with no copy to drift?"
  - "internal/creelengine/machinemail_test.go TestStore_MachineOutbound: does a test fail if a post to another machine starts a per-hub outbound worker, at the scheduler's start or on a post?"
---

Focus entries name where to look; they do not limit what the review may flag.
