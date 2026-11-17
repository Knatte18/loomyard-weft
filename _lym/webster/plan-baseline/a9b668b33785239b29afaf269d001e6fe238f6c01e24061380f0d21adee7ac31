---
format: 5
approved: true
language: go
---

# Plan: A Codex session gets Claude's standing items, user setup, instruction files and subagent terms in its developer instructions

A Codex launch today puts lym's standing items in its prompt file, in the user turn, and sees none of the user's Claude setup.
This plan moves the standing items into Codex's `developer_instructions` with a 128 KiB refusal (D5),
gives Codex subagents Claude's subagent terms and a thread cap (D6),
adds the shared `agents-prompt` target and a hub-reconcile render of heddle's file targets (D7),
hands the user's Claude setup from `claudeengine` to `codexengine` through a `shuttleengine` seam with a shared `@`-import expander (D8),
and makes Codex read every instruction file Claude reads (D9).

## Card Index

1 — claude-imports-leaf — the leaf `internal/claudeimports` expands a Claude instruction file's `@` imports
2 — agents-prompt-target — the shared prompt target `agents-prompt` reaches Claude and Codex sessions; `lym heddle show agents-prompt` is refused
3 — reconcile-renders-file-targets — the hub config reconcile renders heddle's file targets into every live worktree
4 — codex-file-carries-both — `AGENTS.override.md` carries AGENTS.md and the import-expanded CLAUDE.md, and an import edit re-renders it
5 — codex-developer-instructions — Codex standing items ride `developer_instructions`, refused past 128 KiB; the prompt file holds the prompt alone
6 — codex-subagent-terms — Codex subagents get the role's effort, the Codex subagent passage and the `max_subagent_threads` cap
7 — claude-user-setup-source — the `shuttleengine` user-setup seam and claudeengine's global file, above-root CLAUDE.md files and skill listing
8 — plugin-session-start-contexts — claudeengine runs the scope's plugin `SessionStart` hooks for their context
9 — codex-takes-user-setup — the Codex engine takes the user setup, hook runner and sandbox probe as seams, wired in shuttleengines
10 — codex-instruction-files — `project_doc_max_bytes` covers the files Codex reads, and unread CLAUDE.md files are named

## Shared Decisions

- **D7's shipped-entry move is already in the tree.**
  Commit `22b74f24` moved `lym-batching` and `lym-plan-edits-before-gate` to `agents-file` and reworded them so no line names a Claude tool;
  `internal/heddleengine/shipped.json` holds them so, and `TestStore`'s `ShippedEntriesArePinned` pins the target.
  No shipped entry targets `claude-file`, and `claude-settings` holds only Claude Code settings objects, so no card changes the shipped layer.
- **Developer-instructions order.**
  The items follow the order Claude's appended system prompt file holds them: the deny notice with the creel trust sentence, the subagent passage, the standing passages;
  then the user's instruction files, the instruction-file pointers, the skill listing and the plugin contexts (D5).
  Each item carries a name and a way forward, so the 128 KiB refusal names the largest item and how to shrink it.
- **The seams.**
  `shuttleengine.UserSetupSource` is the one route from `claudeengine` to `codexengine`, wired in `internal/shuttleengines` (`PATTERN-shuttle-provider-seam`).
  `codexengine.Seams` carries the user-setup source, the plugin hook runner and the sandbox probe into `NewFromConfig`, so a later test can pass a faked runner and probe, or a runner that lists each hook unrun (a nil runner);
  production passes claudeengine's source and `RunPluginHook`, which keeps each seam reachable from `main`.
- **Hub reconcile render route.**
  `hubreconcile` may not import `internal/lymcwd` (`TestToldGeometryInvariant_AllowlistOnly`), so card 3 adds `fabricengine.NotifyWorktreeMadeAt`, which resolves the worktree and fires the installed heddle seam.
  That adds one function to `internal/fabricengine`, a package outside the record's scope list.
- **Plugin hook processes.**
  A `SessionStart` hook run is logged at its spawn and end and waited on under its timeout.
  It registers no `runproc` record, since `PATTERN-child-processes` asks one of a run's detached or long-lived processes, and a hook run is neither.
  Bound: a hook left by a `Prepare` killed mid-run runs to its own end with no owner, review-held, as card 8 states.
- **Subagent effort.**
  The Codex subagent effort is the effort agents.yaml gives the role on a model it does not list, the counterpart of Claude's fallback effort;
  an effort codex.yaml's `efforts` does not list is left off with a warning, rather than refusing every launch of that role.
- **Text in Go literals.**
  The Codex subagent passage, the skill listing and the pointers name no Claude tool.
- **Constraints carried by sibling parts.**
  The record's constraints on D1, D3 and D12 (the read-only lookup package both nudges share, the stencil edit-directive background, the tolerant manifest record, shuttle's cobra verb and the tier0 parity test and scans) belong to the parts that carry those decisions.
  This part's decisions are D5 to D9, and no card here carries those constraints.
- **Plugin way forward.**
  `session_scope` turns a plugin on exactly when the session preloads one of its skills, so turning a plugin off in the session scope means dropping its skills from the role's skills in agents.yaml or from the spec,
  or turning `session_scope` on while it is off; every way forward that says so names those steps.
