---
round: 1
focus:
  - "internal/dobbyengine/runnotice.go against the removed code in internal/battenshed/notice.go: settle whether the check order (dead driver, usage-limit, api-error, quiet), episode keys, quiet window, api-error idle time, and the idle clocks that subtract sleeps and paused intervals are unchanged."
  - "api-error revive streak in runnotice.go: settle whether it revives on first sight, notices on the next sight of the same episode, and on the third revived episode in a row with no child status write between, and whether the streak survives a dobby re-exec through the state record."
  - "Wait markers and pause handling: settle whether a live wait marker (such as a pull request wait) holds quiet, and whether a paused child, an absent child status or a cancel marker puts a run in no condition."
  - "Sent-notice marker (shedrun.NoticeEpisodeFile, internal/battencli/commitstatus.go): settle whether a sent key is written to the marker path, a repeat tick in the same episode sends nothing, a new episode drops the old key, and the status commit carries the marker only when it exists; check that cleanup removes it with the run directory."
  - "Retry and drop of undelivered notices: settle whether a notice not posted is retried on later probe ticks, dropped with a Warn after three failed attempts, and that the attempt count stays in process."
  - "Scope split between dobby and batten in internal/battenshed/notice.go and innerrun.go: settle that batten's Run-Shed row no longer sends quiet, api-error, usage-limit or running-child driver-dead notices and makes no api-error revive, while state-changed, driver-not-revived, busy-pane and owner-elsewhere notices still behave as before and keep `[batten notice]`."
  - "Repository rules on the new dobbyengine code and its seams (internal/beamcli/dobbydeps.go, internal/battencli/dobbynotice.go): settle compliance with PATTERN-cwd-resolution (no CLI or lymcwd import in dobbyengine, import allowlist), PATTERN-durable-vs-ephemeral-state (only lymdirs names _lym), PATTERN-tolerant-records for the new state fields, and PATTERN-agent-messaging."
  - "Rule and doc text naming the line: settle that the orch role stencil's Notices and stop sections, docs/overview.md and the package docs name `[dobby notice]` for moved notices and `[batten notice]` for the rest, and that no rule, stencil or doc still names a moved notice as a `[batten notice]` line."
  - "Test economy in internal/dobbyengine/runnotice_test.go and the trimmed battenshed tests: settle whether each acceptance criterion's behavior has a tier0 test with injected clocks, whether any new test duplicates another's coverage, and whether removed battenshed tests left a batten-kept behavior uncovered."
  - "Change description and commit message claims: settle whether each claim in the commit message and `_lym/landing/summary.md`, if present, is borne out by the diff, including the removal in internal/testkit/envkit/envkit.go and internal/loomcli/wiring_seams_test.go that the answer key does not obviously license."
---

Seed focus for the first code-review round, set from the board entry, decision record and the diff's file list.
