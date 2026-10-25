# Shortcuts are short forms only: drop attach, pause, resume, start

The top-level command surface carried long-form shortcuts beside the short ones.
`lyx attach`, `lyx pause` and `lyx resume` duplicated `lyx a`, `lyx p` and `lyx r`, and a top-level `lyx start` ran `lyx loom start`, which nobody runs since batten starts every loom run.
Those four root commands are removed; the short forms and `lyx loom start` keep their behavior.

## What changes

- `internal/shortcutcli/shortcuts.yaml` drops `attach`, `pause` and `resume`; the shipped set is `a`, `p`, `prime`, `r`.
- `loomcli.StartAliasCommand`, its registration in `cmd/lyx/main.go`, its test and the sandbox coverage allowlist row for `start` are deleted.
  `newLoomCLI` keeps one caller, and `lyx loom start` keeps its flags, including the hidden `--no-attach`.
- Every refusal, notice, envelope line, help text, doc comment, doc and sandbox suite line that named a removed form now names a surviving command.
  `lyx resume` and `lyx pause` become `lyx tender resume` and `lyx tender pause` with the same arguments, and `lyx start` becomes `lyx loom start`.
  Tests that asserted the old text assert the new text.
- The tender module help and `docs/overview.md` name `lyx p` and `lyx r` as the shortcuts for `lyx tender pause` and `lyx tender resume`.

## Design decisions

- Messages name the full command, not the one-letter shortcut.
  The full command exists independent of the shortcut file and is greppable; a bare `lyx r` in a refusal reads as a typo.
- `lyx prime` stays: it duplicates no other shortcut and the change does not name it.
- The start alias is deleted outright, with no hidden shim for internal callers.

## To check by hand

- `lyx attach`, `lyx pause`, `lyx resume` and `lyx start` are rejected as unknown commands, while `lyx a`, `lyx p`, `lyx r` and `lyx prime` run as before.
- Another in-flight change touching `internal/fabricengine/originreconcile.go` and `internal/adoptcli/adopt.go` carries `lyx resume` strings; a textual conflict there resolves by keeping both changes.
