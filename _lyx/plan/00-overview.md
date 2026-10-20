---
format: 5
approved: false
language: go
---

# Plan: verify-diff-scope

A darn run stops paying for the whole-module suite where its diff does not need it.
Its writer and fix-round gates run the diff's impacted set from the merge base plus the guard-marked scans, and a diff wholly under the hub's configured non-production trees runs only its trees' command at every darn verify site, landing included.
The cards build the pieces bottom-up: the both-sides diff read, the `darn.yaml` keys and classifier, the editor's loader check, the module guard scan, the derived command's move onto `lyx gate test`, the `--guards` flag, the darn gate form, the landing sites, and the prose that describes them.

## Card Index

1 — both-sides-diff-read — add the verbatim, rename-splitting changed-path read darn's classification runs
2 — darn-tree-scope-config — add tree_scope and tree_scope_verify to darn.yaml with their refusals and the pure tree classifier
3 — editor-save-load-check — load a hub-wide editor save through the module's loader, keeping the staging copy on a refusal
4 — module-guard-scan — give guard-scan errors a kind and line and add the module-wide guard scan
5 — derived-command-gate-test — run the derived command's test and guard steps through lyx gate test
6 — gate-test-guards — add lyx gate test --guards
7 — darn-gate-impact-scope — replace darn's whole-diff gate form with the impact-scoped, tree-aware form
8 — darn-landing-tree-scope — resolve darn's Merge-In and Finalize command per call and skip publish_verify for a tree-scoped diff
9 — darn-verify-prose — restate the darn writer stencil, the PATTERN backgrounds and the overview

## Shared Decisions

- The darn gate form is selected by a told tree-scope seam, nil for loom, replacing the `wholeDiff` flag, whose only user was darn; loom's round form is unchanged.
- The derived command's guard steps keep per-package `-run` selection on plain `lyx gate test`, not `--guards`, so the `lyx` a task pane finds on its `PATH` runs them before the flag ships there.
- `go list -e` lets the module guard scan reach an unreadable test file, which it reports as the parse kind; an unreadable package directory surfaces as an enumeration entry with no directory, named in the enumeration error.
- The criterion that the change description names the commands that turn tree scope on and off is met at Describe, outside every card: the change description is the landing's own file, written from the final diff, and no card targets it.
  Card 2 puts both commands, `lyx config darn --set tree_scope.tools=./tools/... --set tree_scope.docs= --set "tree_scope_verify=lyx gate test --tags integration --guards {targets}"` and `lyx config darn --set tree_scope_verify=`, in `internal/darnengine/doc.go`, which Describe reads from that diff.

## verify:

```
go build ./...
lyx gate test --tags integration ./internal/fabricengine ./internal/darnengine ./internal/configreg ./internal/configsync ./internal/configcli ./internal/loomengine ./internal/impactset ./internal/gatecli ./internal/loomshed ./internal/shedrecipe ./internal/loomcli ./internal/landingshed ./internal/webstercli ./internal/testkit/envkit ./cmd/lyx
```
