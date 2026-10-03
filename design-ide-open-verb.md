# lyx ide open: launch VS Code without writing anything

`ide-open-verb` · Notes · feature

Seen 2026-10-03: `lyx ide spawn` configures and launches in one step (title colour, `.vscode/settings.json`, an interactive `tasks.json` rewritten every call, `info/exclude`, the prime's hub workspace file).
On a driven pair it replaces batten's attach-only chain with the interactive one, whose `reed add claude` can start a second Claude in that pair, so the orch opened the part-2 pair with plain `code` instead.
Idea: a `lyx ide open [<slug>]` that only launches and writes nothing — the prime through its existing hub workspace file, a pair through its folder and the `tasks.json` already there — with the same no-argument and hub-root defaults as the reopen note; `spawn` stays the configuring verb.
