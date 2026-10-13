---
format: 5
approved: false
first_card: 10
language: go
---

# Plan: reed window order — overview first, agents in start order, shell last

A reed session's visible windows follow one fixed order: the run overview first, one window per agent in start order, then the shell (Selvage) last, with hidden windows after it.
Cards 1 to 9 built that order, the `overview` anchor, the focus rules, the window navigation and the plain overview panes.
The PR review rejected one fix-round change: deleting `Engine.SetContentRows` retires content sizing, which the decision record's Scope Out keeps valid for an explicit caller.
This generation restores that operation and leaves every other built change as it is.

## Card Index

10 — restore-set-content-rows — bring back `SetContentRows`, its `InstallPins` apply option, its tests and its doc paragraph

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
