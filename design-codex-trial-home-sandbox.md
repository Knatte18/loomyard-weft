# Codex trial: home, trust, sandbox and refusals

`codex-trial-home-sandbox` · Task · research, codex, shuttle · running · Record-Write

## Goal

Test parts 1 and 2 of project `codex-engine-trial` against real Codex sessions launched through `lym shuttle`: home and trust, sandbox and refusals, as `internal/shuttleengine/codexengine/doc.go` describes them.

## Cases

Each case runs once, on `luna[medium]`, in this task's worktree or in a throwaway directory the case makes under it.

1. A launch in this worktree: `CODEX_HOME` is the lym Codex home; `auth.json` and `sessions` link to the operator's home; `config.toml` is rendered whole and trusts the worktree root and the main checkout root.
2. Two launches started together: the file lock keeps `config.toml` whole and both roots trusted.
3. A recorded root whose folder is removed is dropped at the next launch.
4. A regular `auth.json` in the lym home is named in a Warn and left in place.
5. The launch line carries `-a never -s workspace-write`; the session writes under the anchor's `.lym` and is refused a write outside its writable roots.
6. A committed `.codex/rules` refuses the launch; a committed `.codex/config.toml` is named in a Warn.
7. The folder-trust prompt, raised by a root removed from `config.toml` by hand before Codex reads it, is accepted by the engine's backstop.

## Deliver

- A report at `docs/research/codex-engine-trial-home-sandbox-2026-11-03.md`: per case, what was run, what was seen and whether it matches the package doc.
- Each lym fault as a bug note on the board (`kind: note`, label `bug` and the module labels), one finding per note.
- The operator's own `~/.codex` is never written: check its `config.toml` checksum before and after.
