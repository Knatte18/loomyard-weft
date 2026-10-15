---
format: 5
approved: true
language: go
---

# Plan: discussion-relevant-list

The loom Discussion-Write lead writes `_lyx/discussion/relevant.md`, a list of the files and members it read and found relevant, each with one line of why, beside the decision record and the support log.
The discussion spec pins its format, a `loomengine` accessor declares its path, `discussionparser` checks a present list, and the loom discussion gate and `lyx loom validate-discussion` run that check through one `loomshed` function.
The Discussion stencil and the twill relaunch part ask for the file, and the Plan stencil reads it first as a head start, never as scope.
Darn, `Validate`, `lyx loom decision add` and every reviewer are untouched.

## Card Index

1 — discussion-spec-relevant — pin relevant.md's format and the plan writer's inputs in the discussion spec
2 — relevant-accessor — declare relevant.md's path with `DiscussionRelevant`
3 — relevant-check — add `CheckRelevant`, the format check of a present list
4 — loom-gate-and-verb — run the check in the loom discussion gate and validate-discussion through `ValidateLoomDiscussion`
5 — twill-relaunch-files — a relaunched Discussion writer finishes all three files
6 — discussion-writes-relevant — the Discussion-Write lead lists and writes relevant.md
7 — plan-reads-relevant — Plan-Write reads relevant.md first when it exists

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
