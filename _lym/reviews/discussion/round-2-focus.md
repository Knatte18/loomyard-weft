---
round: 2
exclude_lenses: []
focus:
  - "Host-side attach to an area hub (the record's D6, D7 and D13 `window`, against `internal/reedcli/attach.go`): which hub config, hub state or agent-chosen binary does the host process read or run before it is restricted, and can the supervisor serve its connects under Yama `ptrace_scope` 1 when the client is not its descendant?"
  - "Every socket a host process or another hub connects to (supervisor, tmux, creel; D3, D6, D8): can an agent unlink or replace it, and how does the caller know who answers?"
  - "Every host-side path that reads, executes or signals something agents can write (Go build and module caches, repository code run by `lym area deploy`, `runproc` records read by `lym area stop`, the Slack relay's setup, maps and outbox, derived creel paths from the machine record): what bounds each, and does the record's Goal or non-claims say so?"
  - "Host verbs that reach a hub without its cwd resolving to it (`lym machine start`, machine-record sweeps, cross-hub creel geometry): which of them run git or read config over an area hub?"
  - "Identity across the boundary (D3's inside test, D8's creel refusal, D9's `cap-memory` bound): how is a peer treated when its pid is gone or unreadable, when it carries a filter but no supervisor ancestry, or when it is a host-side child of the supervisor, and can a daemon read what the test needs?"
  - "Supervisor lifecycle (D3, D4, D9): the server's environment, the order of `no_new_privs`, Landlock and the filter in a cgo build, stopping an area whose supervisor died, and ending a capped run scope's tree from inside the area. Is each stated, and is each tool behaviour it rests on pinned?"
  - "The grant lists (D2, D5, D6, D10, D15) against each other: do `lym area init`'s lists, the start check over derived write grants, codex's home and the AppArmor profile's hash agree?"
---

Every lens that ran reported findings at MEDIUM or worse, so none is excluded.
