---
format: 5
approved: true
language: go
---

# Plan: role language skills

Every spawned role that writes or reviews code loads the hub's language skills after its own skill list: the darn writer, the Webster master and recovery strand, the merge conflict session at Merge-In and at Finalize, and the Burler seats of a code round.
A new hub-wide config module, `language`, holds the ordered skill list in its one Guard key `skills`, with an empty template default, so lyx hard-codes no language.
Each engine takes the list as a told value and merges it after its own list through one shared helper that drops repeats; each wiring that builds a code role loads the module once per process.
The darn writer also gains `scribe:testing`.

## Card Index

1 — language-config-module — add the hub-wide `language` module, its loader and the shared skill-merge helper, and register it
2 — darn-writer-skills — the darn writer gains `scribe:testing` and loads the told language list after its own
3 — webster-role-skills — the Webster master and recovery strand load the told language list after `roleSkills`
4 — conflict-session-skills — the merge conflict session loads the told language list at Merge-In and at Finalize
5 — loom-wiring — `lyx loom`'s wiring loads the module and tells the list to darn, Webster and landing
6 — webster-wiring — `lyx webster`'s hub wiring loads the module for `run` and `recover-batch`
7 — burler-seat-skills — a Burler half marked as a code round loads the told language list on every seat
8 — burler-profile-key — the recipe and `lyx burler run` profiles accept `language-skills`
9 — recipe-code-rounds — the Webster, Merge and Darn review and fix rows carry `language-skills: true`

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
