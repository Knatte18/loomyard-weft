---
format: 5
approved: true
language: go
---

# Plan: reed-tmux-conf — an opt-in operator tmux config for reed's hub servers

Give the operator one `reed.yaml` key, `tmux_conf`, naming a tmux config file that reed sources into a hub server on the boot that starts it, before reed's own pins.
Reed's servers stay hermetic by default; a named file is validated at boot, sourced exactly once per server start even under concurrent boots, and may not end reed's session or server, and a session it creates is invisible to reed and never keeps the server alive.
Windows parses the key and does nothing with it.

## Card Index

1 — tmux-conf-config-key — add the `tmux_conf` key to `Config` and both templates, and unset its env variable in `tmuxkit.Main`
2 — tmux-conf-path-validation — refuse a bad `tmux_conf` on every POSIX boot, in the shell refusal's place and style
3 — tmux-conf-boot-source-guard — source the file once at the server's start, before the pins, under the claim and the session guard
4 — reed-session-filter — make every session read, in Go and in the status bar, see only the sessions that count as reed's

## Shared Decisions

### Decision: window-form-target-in-chains

- **Decision:** A `set-option` chained after `new-session` in one tmux invocation targets the new session as `exactSessionWindowTarget`, the `=<name>:` form, never the bare `=<name>`.
- **Rationale:** `set-option` resolves a pane target, and the probe showed a bare `=<name>` fail with `no such session` while the window form resolved, which is the distinction `exactSessionWindowTarget`'s doc comment already records.
- **Applies to:** card 3

### Decision: read-back-decides-pins

- **Decision:** The guard invocation's exit status is only ever a warning; pin failure and a dead session are detected by the separate read-back that follows it.
- **Rationale:** One exit status carries the claim, the source and both pins, and a typo in the operator file exits non-zero while both pins apply, so only a read-back separates the two, per the decision record's parent addition.
- **Applies to:** card 3

### Decision: filter-off-on-windows

- **Decision:** The session filter is passed to `list-sessions` off Windows only; the status-bar format carries it on every platform like every other geometry pin.
- **Rationale:** No Windows boot ever claims a server, so the filter would admit every session there anyway, and psmux's `-f` support is unverified; a rejected format in the status bar degrades non-fatally, a rejected `list-sessions` flag would break every listing.
- **Applies to:** card 4

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
