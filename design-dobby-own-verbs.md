# Dobby gets its own lyx dobby verbs, and lyx orch start stops starting it

`dobby-own-verbs` · Note · enhancement, orch, workspaces

## Seen (operator, 2026-10-25)

`lyx orch start` does four jobs: it starts every dobby this machine owns, reconciles the hub's branches against origin, ensures the overview window, and only then starts the orch session and its watcher.
Dobby's own verbs sit under `lyx beam` (`lyx beam start`, `lyx beam status`), and `lyx prime` names the top beam although the verb runs in any beam.
`--handoff` and `--adopt` are never used; `--continue` is used often and is long to type.
The command surface carries long-form shortcuts beside the short ones and a top-level `lyx start` for `lyx loom start`, which nobody runs since batten starts every loom run.

## Design (operator, 2026-10-25)

A beam, the prime included, is one unit of two parts that always run together: its daemon, dobby, which starts, drives and lands the beam's runs, and its LLM, the orch, which handles stops and questions and is always there to talk to.
Dobby already sends its notices to the orch, and an idle orch spends nothing until a message wakes it.
Both run on the machine that owns the beam.

- `lyx beam start [slug]` brings a beam up, the current worktree's beam with no slug: it starts the dobby, reconciles the branches, ensures the overview window, and starts the orch with its watcher.
  The orch resumes its recorded session when one exists, else starts fresh from the last handoff note, else from the start stencil.
  A recorded session that cannot be resumed falls back to a fresh start from the note, never to an error.
- `lyx beam stop [slug]` stops both, and `lyx beam status` shows both.
- A fresh orch comes from `lyx orch refresh`.
- `lyx orch` keeps `stop`, `status`, `distill`, `refresh`, `withdraw` and `rules`.
- Removed: `lyx orch start` with `--continue`, `--handoff` and `--adopt`; `lyx orch cleanup`, since `lyx cleanup` already removes old handoffs; `lyx prime`; `lyx start`.

Shortcuts are short forms only:

| Shortcut | Runs |
|---|---|
| `lyx s` | `lyx beam start`, then `lyx reed attach` |
| `lyx a` | `lyx reed attach` |
| `lyx p` | `lyx tender pause` |
| `lyx r` | `lyx tender resume` |

The long forms `lyx attach`, `lyx pause` and `lyx resume` are removed.
