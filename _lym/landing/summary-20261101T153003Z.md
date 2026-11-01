# Guard steers name the version probe and a package-scoped compile check

Agents in lym sessions kept running commands the `PreToolUse(Bash)` guards refuse, and each refusal costs a turn at the session's full context.
Recent fork transcripts held 61 such refusals: 48 Python and 13 raw `go`.
Of the 48 Python refusals, 41 were `python3 --version` probes, 24 of them placed before a `perl -0pi` or `sed -i` edit in the same command;
only 6 tried to run a Python script and 1 to parse JSON.
Every raw `go` refusal was `go build ./...`, `go build ./internal/...` or `go vet ./internal/...`, used as a compile check after an edit.
The probes and compile checks are habits the old steering never addressed:
it said "lym agents never run Python", which a fork need not read as covering a version check,
and it named only `lym gate test`, never a quick compile check that the guard lets through.

## What changes

- **Python row** (`internal/shellguard/rows.go`).
  The notice now reads "python and python3 are refused in every lym session, a version check included, so never probe for Python."
  The steer adds "parse JSON with jq and text with grep or awk" ahead of the edit rule it already carried.
- **Raw go row.**
  After its first sentence, the steer and notice add "A compile check names its packages, as in go build ./internal/x or go vet ./internal/x, never ./... or all."
  `compileCheckSentence` names each form only while no `gate.yaml` `refuse_raw` prefix refuses it, and drops the sentence when every form is refused, so the steer never offers a refused command (`PATTERN-refusal-way-forward`).
- **Plan spec.** `contracts/specs/loom-plan-spec.md`'s compile criterion said "`go build ./...` succeeds", which modelled the refused form.
  It now reads "the whole module builds".
- **Docs.** The `shellguard` and `claudeengine` package docs and the shuttle row of `docs/overview.md` describe the new steer content.

## Design decisions

- **Steering, not a looser guard.**
  Letting `python3 --version` through would tell an agent that Python is present and invite the script that follows.
  Naming the probe as refused removes the reason to run it.
- **Package-scoped compile checks are the way forward, not only `lym gate test`.**
  The raw go pattern already lets `go build ./internal/x` and `go vet ./internal/x` through, and they cost far less than a gate run.
  Two pattern-table rows pin that both offered forms pass the raw go row.
- **The edit-plus-probe case is covered elsewhere.**
  Its shell edit is now refused on its own by the shell-edit row from `edit-with-edit-tool`, which landed after these transcripts and steers edits to Edit.
- **No heddle line in this change.**
  The record's third idea, a trial line in the rendered `CLAUDE.local.md`, is heddle text written into the Bolt checkout's `heddle.json` by `lym heddle set`, not code in this worktree.
  The record ties it to the round of the `batching-in-claude-local` trial, so it is left to that run.
  It is no own-entry step, so no drop is recorded for it.

## Tests

- `TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks` (tier0) checks the compile-check sentence across four `refuse_raw` lists: none, `go test`, `go build` as well, and `go`.
- `TestRows_PatternsDenyAndAllow` (tier1) gains `go build ./internal/x` and `go vet ./internal/x` as allowed by the raw go row.
- `TestBuildSettings_MatchesGolden`'s `settings-golden.json` is updated to the new steer and notice text.
