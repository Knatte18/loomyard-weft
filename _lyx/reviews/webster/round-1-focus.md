---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/usage.go: does the backward chunked reader parse only complete lines, read an entry spanning a chunk edge whole, and pick the later of the latest assistant entry and the latest non-sidechain compact_boundary as the card 1 intent states?"
  - "internal/orchengine/watcher.go tickIdle: does the re-read after all gates apply the threshold matching the trigger (hard cap for hard, soft threshold for soft), skip the re-read for a requested cycle, and avoid upgrading a hard trigger into a soft firing on the same tick?"
  - "internal/orchengine/watcher.go tickCompacting and restart path: does completion require a boundary with BoundaryAt at or after PhaseEnteredAt plus an idle probe pass, does timeout return to idle with 'compaction timed out' and set LastDeferral, and is an unconfirmed injection re-typed only on idle with no qualifying boundary read?"
  - "Compact mode versus clear mode: is clear mode's behaviour unchanged (shared test env pins cycle_mode clear), are DEFER handling and handoff writing absent in compact mode, and is a requested cycle never held by the D8 deferral hold?"
  - "internal/orchengine/config.go LoadConfig: does it still go through configengine.LoadOrTemplate, refuse permission_mode other than empty or bypass with the fix named, refuse an unknown cycle_mode naming both values, and resolve empty permission_mode to bypass on the returned Config?"
  - "Notice queue and batten wiring (cards 7 to 9): does only the orch watcher type from the queue, is delivery idle-gated, does battenshed queue only through the injected Notify seam without importing orchengine, and is the queue path declared once in internal/orchcli/paths.go?"
  - "Cards 10 to 12: do the stencils drop watcher re-arming and teach the notice rule, does adopt let a holder through only when the pid is proven reused, and do the smoke tests assert the compact cycle, the bypass footer and a large-session adopt as the cards state?"
  - "Per-card mechanical check: for each card's groups, did the type-specific mechanical checks from the plan spec (Rename AST-script plus grep per target, assert-no-callers for Delete) actually run and pass, and are the verify commands in the plan overview (build, test, integration test, smoke vet) satisfied?"
  - "Comment conventions: do new and changed doc comments, doc.go sections and PATTERN entries follow CLAUDE.md's rules (semantic line breaks in .md files, docs landing in the same commit, no perishable tallies) and the surrounding code's Go doc style, with no inferred line width?"
---

Seed focus for round 1, derived from the plan's cards and shared decisions.
No finding is pre-stated; each entry names a place and the question to settle.
