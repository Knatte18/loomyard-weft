---
format: 5
approved: true
language: go
---

# Plan: creel delivers every post to a Claude strand through hooks

Creel hands a Claude strand its mail through hooks shuttle renders into the session's settings: a visible PostToolUse block at the next tool call, a Stop block at the turn's end that records no turn end while it blocks, and an `asyncRewake` waiter that the daemon rings to wake an idle session, with a typed fixed bell where no waiter reaches.
Only the main thread takes mail, every line a claim hands out fits one hook field, the pane path for posts stays only for a strand whose engine has no hook delivery, `post --interrupt` stops a turn and then posts, sender names are quoted, and every Claude session shuttle launches gets the trust and reply sentence through one prompt file.
The cards build bottom-up: the reed mark and shuttle launch field, the creel line format and caps, the Stop count, the seam's turn reading, the ring and the bell, the provider capability and the hook verbs, the rendered hooks, `post --interrupt`, the trust sentence, the cross-package docs and the real-Claude smoke test.

## Card Index

1 — reed-creel-hooks-mark — reed carries the opaque CreelHooks mark through add, replace and resume
2 — shuttle-launch-creel-hooks — a launch's CreelHooks reaches its reed strand
3 — creel-quoted-sender — creel quotes every sender name in the lines it writes
4 — creel-line-cap — post refuses a line past the 8 KiB claim budget and the claim skips old over-cap lines
5 — loom-decision-notice-cap — LOOM's decision notice fits creel's line cap
6 — creel-stop-continuation-count — the claim knows its boundary and caps a turn's Stop blocks at 6
7 — shuttle-idle-no-turn — shuttle's idle probe reports whether a turn is in progress and when the last one ended
8 — creel-delivery-seam-fields — the delivery seam tells a hook strand from a pane-mail strand and reads the turn
9 — creel-pane-path-by-engine — the pane path types posts only into a strand whose engine has no hook delivery
10 — creel-hook-waiter-ring — the daemon rings an idle hook strand's waiter once per claimable set
11 — creel-bell — a typed fixed bell wakes an idle hook strand no waiter reaches
12 — shuttle-hook-capability — the capability parses hook input and renders the visible block per event
13 — creel-hook-verbs — hook-read takes the Stop and filters subagents; hook-wait is the idle waiter
14 — claude-creel-hooks-render — Claude's settings merge the Stop delivery into the turn-end record and add the waiter
15 — creel-post-interrupt — post --interrupt stops the turn, then posts; creel's help carries the reply rule
16 — claude-trust-sentence — every Claude session gets the trust sentence through one prompt file
17 — creel-hook-docs — the orch stencil and cross-package docs describe the hook delivery
18 — creel-hooks-llm-smoke — real Claude sessions prove the hook delivery end to end

## Shared Decisions

- **The line cap meets the added body-cap decision without lowering `maxBodyBytes`.**
  The added decision "A post body is capped at the hook field budget" lowers the post cap to the 8 KiB claim budget, while D1 and the acceptance criteria keep the 64 KiB cap for a reply to EXTERNAL, which only a waiting `post --wait-reply` receives and no claim hands out.
  Card 4 caps every post and strand reply by its whole `MessageLine` against the 8 KiB budget, which is stricter than a body cap of the same size, and leaves `maxBodyBytes` at 64 KiB as the EXTERNAL-reply cap alone, so both decisions hold.
- **The engine decides a strand's path, read from reed's provider tag.**
  A strand whose provider is empty or names an engine with no tool-boundary renderer is a pane-mail strand and keeps the notice and the reminder, so a hand-added strand, a Codex session included, still gets its mail; a strand marked `CreelHooks` gets the ring and the bell; a Claude strand without the mark has neither and keeps its mail for `lym creel read`.
- **The ring's hold rests on the hook reading's turn-end time.**
  A woken turn records no `UserPromptSubmit`, so shuttle's idle probe reports the time of the turn end its no-turn reading rests on, and the daemon holds a strand's no-turn reading false from a ring until a later turn end; a reading the pane decided is taken as it reads.
- **The LLM-tier scenarios use shuttle's own launch.**
  Every real-Claude scenario starts its session through a shuttle runner with claudeengine told the hub's config directories, so the settings and launch line under test are the ones production renders.
