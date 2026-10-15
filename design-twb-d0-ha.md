# Hibernate the machine on a trigger, such as a task landing

`twb-d0-ha` · Note · feature · running · Loom-Preflight

## Seen (2026-10-13)

From the phone, the operator asked the orch to hibernate the machine; it ran `systemctl hibernate` and the machine went down.
Doing it ad hoc from the orch was not a good idea (operator), but the operator wants a way in lyx to trigger a hibernate.

## Idea (operator, 2026-10-13)

The operator can trigger a hibernate through lyx, for instance when a task is done.

## Direction (operator, 2026-10-14)

No LLM session runs the hibernate itself: a session that does is left hanging mid-call when the machine goes down.
Instead the orch sets a hibernate flag on the machine, which queues the request.
The flag asks for a pause (as `lyx pause`) on every instance, and once the pause is confirmed done on all of them, lyx's Go code runs the hibernate on its own.

## Points to settle

- The trigger: a one-shot "hibernate when `<slug>` lands" (or when every run in flight is done), set from the phone through the orch, and perhaps a plain `lyx hibernate` verb.
- Who fires it: batten after Land (and its deploy), so nothing is mid-write.
- Before hibernating: no run mid-turn and no Land in flight; the hub's runs are paused or idle, as the operator already hibernates with runs live.
- Platform: `systemctl hibernate` on Linux, the only platform the operator runs lyx on; elsewhere the verb says it is unsupported.

## Pause and resume shortcuts

Add short shortcuts for pausing and resuming to `internal/shortcutcli/shortcuts.yaml`, next to the existing `lyx a` shortcut.
The file maps a shortcut name, mounted as `lyx <name>`, to one argv (a list of words) or a list of argvs run in order; a shortcut names lyx words only, and is added only by a commit to this file.
It holds `attach` and `a` (both `[reed, attach]`), `prime` (`[[orch, start], [reed, attach]]`), and, since the `tender` task, `pause` (`[tender, pause]`) and `resume` (`[tender, resume]`).
So `lyx pause` and `lyx resume` are already shortcuts; what this section adds is a one-letter or short form beside them, the way `a` sits beside `attach`.

Proposed names, for the operator to confirm: `p` for `[tender, pause]` and `r` for `[tender, resume]`.
Whether to add them at all, and which names, is the operator's call.

