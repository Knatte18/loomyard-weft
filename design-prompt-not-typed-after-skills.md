# Start prompt held when a long strand name fills the input box top border

`prompt-not-typed-after-skills` · Task · bug, shuttle

Folds issues #195 (symptom) and #196 (cause), and the orch's own sighting.

## Seen

A spawned row session gets its skill bootstrap, answers it, and then sits at an empty prompt: its task prompt never arrives.
`lym loom status` keeps an empty `error` and `wait`, and no notice reaches the parent.

- 2026-11-23, this hub: `ly:godocreflow-worsens-findings:code-burler-fix` sat idle 30 minutes at Code-Burler-Fix until the orch's creel message woke it.
- #195, another hub: a Plan-Bouncer seed session stalled twice in a row on the same row, lym's own respawn included; its `events.jsonl` shows `SessionStart`, one `UserPromptSubmit` and `Stop` within three seconds, and nothing more.
  `lym loom goto --to Plan-Bouncer` was refused as `shed_busy`.

## Cause (#196)

shuttle's idle check does not see the input box when the strand name fills the box's top border, so it holds the start prompt:

```
level=WARN msg="shuttle: could not type the segment color" color=orange error="shuttle: the session is not idle: strand \"ebb49562…\": no input box is on screen; a menu or picker may be open; retry once the session is idle. The pane's last lines:
● ok scribe:prose, scribe:code-quality,
  scribe:testing
✻ Sautéed for 5s · done 11.08
─ dop:models-torque-drag-adapter:plan-burler-review ─
❯ 
─────────────────────────────────────────────────────
  ⏵⏵ bypass permissions on (shift+tab to cycle) · …"
level=WARN msg="shuttle: the start prompt was held busy, sending it again" retry=1 reason="a turn is running"
```

The top border holds the strand name with one `─` on each side in a pane about 53 columns wide.
`isTopBoxRule` in `internal/shuttleengine/claudeengine/session.go` takes a labelled top rule only when it starts with three rule glyphs or ends in `… ─`.
It hits long slug and role names in narrow panes; every retry hits the same check.
Whether this hub's sighting had the same cause or the 529 Overloaded errors of that morning is for the run to judge.

## Workaround

The parent posts the session's `promptLine` (from its `run.json`) to the strand by creel, or the operator widens the pane.

## Expected

- The idle check finds the input box by the `❯` prompt line and the borders around it, whatever text the top border carries and however few `─` are left beside it.
- A start prompt still held after its retries raises a notice to the parent naming the strand; a respawn on the same row does not repeat the failure.
