# Idle check misses the input box when a long strand name fills its top border, holding the start prompt

`idle-check-long-strand-name` · Note · bug, shuttle

Imported from [issue #196](https://github.com/Knatte18/loomyard/issues/196).

Import note: the issue names files under `.lym/`, which is deleted with its worktree, so only what the issue quotes of them reached the board: `.lym/shed/`.

Root cause of #195 (a spawned session gets its skill bootstrap but never its task prompt).

## Seen

shuttle's idle check does not see the input box when the strand name fills the box's top border, so it holds the start prompt and the segment colour, and the session waits at an empty prompt.
The run's step log (`.lym/shed/<slug>/steps/<trace>.stderr.log`):

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

The input box is on screen.
Its top border holds the strand name `dop:models-torque-drag-adapter:plan-burler-review`, which leaves one `─` on each side in a pane about 53 columns wide.
The earlier stall on the same run shows the same shape: `── dop:models-torque-drag-adapter:plan-bouncer-seed ─`.

It hits a run whose slug and role names are long in a narrow pane: here slug `models-torque-drag-adapter` with the roles `plan-bouncer-seed` and `plan-burler-review`.
Runs in other hubs, with shorter slugs, do not hit it.
Three sessions in a row on this run stalled, and the retry hit the same check each time.
No notice reached the parent, since the run's `error` stayed empty.

I read this from the log and the pane text, not from lym's source, so the exact rule in the idle check is inferred.

## Workaround

- The parent orch posts the run's `promptLine` (from the session's `run.json`) to the strand by creel; the session takes it as its next prompt.
- The operator widened the run's panes so the name leaves room in the border.

## Expected

The idle check finds the input box by the `❯` prompt line and the borders around it, whatever text the top border carries and however few `─` are left beside it.
A start prompt still held after its retries raises a notice to the parent, naming the strand.

## Related

This may be the cause of `prompt-not-typed-after-skills`, which holds issue #195.
`isTopBoxRule` in `internal/shuttleengine/claudeengine/session.go` on main (fetched 2026-11-23) takes a labelled top rule only when it starts with three rule glyphs or ends in `… ─`, so `─ dop:…:plan-burler-review ─` is not seen as the top rule.
