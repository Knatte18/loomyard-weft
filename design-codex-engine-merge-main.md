# Merge main into the codex-engine beam and resolve every conflict

`codex-engine-merge-main` · Task · feature, codex, shuttle · running · Drift-Await

## Build

- Merge `origin/main` into the `codex-engine` beam branch.
- Resolve every conflict so that both main's changes and the beam's Codex work stand.
- Fix whatever the merged tree breaks in the build and the gate, including code that main changed under the beam's new code.

## Bounds

- No new behaviour: the task only reconciles the two sides.
- The project's Sketch (`lym board get P2 --record`) is the authority on what the beam's code must keep doing.