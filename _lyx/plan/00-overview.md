---
format: 5
approved: true
language: go
---

# Plan: webster-audit-gaps

Close the six webster audit and recovery gaps the decision record lists: a pathless audit finding no longer breaks `recover-batch` (#513), the read-only classifier tokenises `lyx`, `git` and `sed` segments shell-style and admits the quoted, globbed, tilde, stderr-redirected, `sed -n` and `--json` help forms (#514, #518) with a pin on the commands that shadow the global `--json`, every admitted git reader form carries a row rejecting a writing flag, a failed Merriam-base read carries one way forward on every path, `begin-batch`'s existing-report remedy follows recover-batch's own order and carries `recovery_exhausted`, and the rebaseline and shell-wait tables gain their thin rows.
Every scenario lands as a row or an assertion in the table that already covers its surface; the one new top-level test is the `--json` pin, which has no table, and it raises its package's budget row in the same card.

## Card Index

1 — pathless-entries-never-resolve — a class-prefixed uncheckable entry is recognised beside its writer and skipped by the three evidence readers before any link resolution
2 — classifier-shell-words — `lyx`, `git` and `sed` segments are split into dequoted shell words, `2>&1` and `2>/dev/null` are dropped before the redirection scan, and `sed -n <lines>p` joins the readers
3 — classifier-json-help-form — a trailing `--json` is a help form of a `lyx` segment
4 — own-json-flag-pin — a walk of the real command tree pins the set of commands declaring a `--json` of their own
5 — git-reader-flag-rows — one row per git reader form rejects a writing, executing or config-touching flag
6 — merriam-base-one-way-forward — `MerriamBase`'s error carries no way forward and each caller words its own
7 — begin-batch-exhausted-remedy — the existing-report remedy words the exhausted, retry, dead, and stuck-or-failed cases in recover-batch's order and the verb carries `recovery_exhausted`
8 — thin-table-rows — the auto-rebaseline warning rows and the ended-shell rows the landing review called thin

## Shared Decisions

- The record's bound for D3 names `lyx shed status` as the only command declaring its own `--json`; the tree mounts the generic shed `status` verb under every recipe as well, so Card 4 pins the set the walk reports, each a reading verb, and Card 3's bound says so.
- The classifier keeps its signature, `IsReadOnlyCommand(cmd string) bool`, so websterengine's injected `ReadOnly` seam is untouched; only unexported segment rules change their word type.
- D4's review of the listed git flags finds none that writes, so Card 5 adds rows only; a flag found writing during the card is removed there.
- The shell-wait rows live in `internal/shuttleengine`'s `TestWait_RecordsEndedShells`, the table that records `EndedShells`, so that package joins the verify set.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
