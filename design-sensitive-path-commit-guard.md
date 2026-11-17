# Refuse a commit that adds a secret file

`sensitive-path-commit-guard` · Task · feature, webster, fabric · running · Drift-Await

## Seen (orch, 2026-11-17)

No lym commit path refuses a secret file: `record-batch` and fabric's commit verbs commit whatever the agent staged.
Unattended Codex and Claude sessions now commit on their own.
Source: the survey F17 (`orchestrator-survey-ideas`), from ccswarm, which blocks `.env`, `*.pem`, `id_rsa`, `credentials*` and `secrets.y*ml`.

## Idea

- A Go check in `record-batch` and fabric's commit path refuses a commit that adds a path matching a deny-list of secret-file patterns.
- The list ships with lym, and a hub may add patterns in its config.
