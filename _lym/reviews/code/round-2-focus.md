---
round: 2
focus:
  - "internal/creelengine/crosshub.go `widestStampedLine` and `validateStampedLine`: does the widest stamp the pre-post length check measures include the longest legal owner/repo that `HubStamp` now leads with, and does a body at the budget still fit on the receiving hub?"
  - "internal/creelengine/journal.go `applyInbound`, `applyCarried` and `carryRecords`: does each fold keep `SendingRepository` on the message, and does a journal test pin it through replay and through a retention carry?"
  - "internal/creelcli/cli.go `messageFields`: is `sending_repository` emitted when set and omitted when empty, and does a test assert both?"
  - "internal/configreg/configreg.go creel module registration, `repositoryOfShortname`, and `hubregistry.LookupRepository`: does a test fail if the `TrustedHubsRepositoryReinterpretation` registration is dropped, and are the recorded, unrecorded and ambiguous shortname cases covered?"
  - "internal/creelconfig/reinterpret.go `RewriteShortnamesToRepositories`: which resolve errors drop a trust entry, and is the reinterpretation still recorded as applied after a transient failure such as an unreadable per-user state directory?"
  - "internal/hubregistry/repository.go `RepositoryName`: can an owner or repo segment with characters outside GitHub's name set reach the agent-facing stamp?"
---

Entries name where to look and the question to settle; the review applies its own severity.
