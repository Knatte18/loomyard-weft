# Restore six caller-binding rules to PATTERN.md

The earlier PATTERN cleanup moved package-local rules into their owning package's `doc.go`.
Six of those rules bind code outside the owning package, and no test catches a break, so an agent working in another package no longer saw them.
This puts them back in `PATTERN.md` as one-line overview entries, so every spawned agent gets them in its prompt.

## What changes

`PATTERN.md` gains six entries, each under the topic heading its subject fits:

- `PATTERN-sparse-hub-base` under CLI and config.
- `PATTERN-parent-lock` under Fabric and git.
- `PATTERN-transient-stop` and `PATTERN-friction-capture` under Shed and loom.
- `PATTERN-agent-name` and `PATTERN-wait-mark-display-only` under Agents and prompts.

Each owning package's doc now cites its restored ID where it states the rule: `internal/hubreconcile`, `internal/parentlock`, `internal/shedengine`, `internal/friction`, `internal/reedengine` and `internal/agentname`.
`internal/friction/doc.go` also states again that `lyx loom start` vouches for the resume it spawns, so a deliberate resume leaves no crash-resume note.
That clause had survived only as a comment in `internal/loomcli/start.go`.

Only comments and `PATTERN.md` change; no code behavior does.

## Design decisions

- **Overview lines only, no `[background]` link.**
  The package doc already holds each rule's bounds and enforcement, and a background file would be a second copy.
  The entry names the package whose doc holds the detail, and the doc cites the `PATTERN-` ID, so a grep ties the two together.
- **`transient-stop` and `friction-capture` are restored too.**
  The audit was unsure about them.
  They meet the same test as the other four: no scan covers code outside the package, and review is the only guard.
- **`(test)` marking follows enforcement.**
  Only `PATTERN-friction-capture` carries it, as its removed line did.
- **Left out of PATTERN.**
  `hub-wide-config`, `recipe-owns-failures` and `batcher-registry` stay in their package docs, as the audit judged them fine.
  Tripwire tests for the restored rules are out of scope.

## Cost

The overview grows by about 1 KB in every spawned agent's prompt, within the `internal/pattern` headroom line.

## To check by hand

- Run `lyx gate test ./internal/pattern` to confirm the real-file check passes with the six new entries; this was not run here.
