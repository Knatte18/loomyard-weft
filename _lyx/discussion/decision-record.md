# Decision record — Shortcuts are short forms only

## Goal

The top-level shortcuts are short forms only: `lyx a`, `lyx p` and `lyx r` stay, the long forms `lyx attach`, `lyx pause` and `lyx resume` go, and the top-level `lyx start` goes while `lyx loom start` stays.

## Scope

- In: `internal/shortcutcli/shortcuts.yaml` drops `attach`, `pause` and `resume`.
- In: the root alias `loomcli.StartAliasCommand` and its registration in `cmd/lyx/main.go` are removed, with the sandbox coverage allowlist row for `start`.
- In: every refusal, notice, help text, doc comment, doc and sandbox suite line that names a removed form names a surviving command instead.
- In: tests that assert a removed form's text or registration follow the change.
- Out: `lyx prime` stays; it is not a long form of another shortcut.
- Out: `lyx loom start`, `lyx tender pause`, `lyx tender resume` and `lyx reed attach` keep their behavior.
- Out: `_lyx/raddle/`, regenerated at landing.

## Decisions

### D1: A message that named a removed form names the full command

Decision: a refusal, notice, envelope line or help text that named `lyx resume` or `lyx pause` names `lyx tender resume` or `lyx tender pause`, with the same arguments; one that named `lyx start` names `lyx loom start`.
Rationale: the full command is the stable, discoverable name that exists independent of the shortcut file, and agents and operators both read these messages; a one-letter shortcut in a refusal is hard to grep and reads as a typo.

### D2: `lyx prime` stays

Decision: `prime` stays in the shortcut file.
Rationale: the board entry removes long forms that duplicate a short shortcut; `prime` duplicates none and the entry does not name it.

### D3: The tender help names the surviving shortcuts

Decision: the tender module's help and `docs/overview.md` name `lyx p` and `lyx r` as the shortcuts for `lyx tender pause` and `lyx tender resume`.
Rationale: the shortcuts still exist, so the module help keeps pointing at them.

### D4: The start alias is removed, not hidden

Decision: `StartAliasCommand` is deleted, along with its test and the comments that name it; `newLoomCLI` keeps one caller.
Rationale: the board entry removes the command, and no shim stays for internal code.

## Constraints

- `PATTERN-cli-cobra`: the root keeps one cobra subtree per module; removing a root child leaves the help-tree and registration guards green.
- `PATTERN-refusal-way-forward`: every refusal still names a way forward that exists, and its test reaches it.
- Docs land in the same commit as the code, per `CLAUDE.md`.

## Auto-mode assumptions

- Messages read by agents may name `lyx tender resume`; the shell guard denies no `lyx tender` verb.
- The sandbox suite markdown under `tools/sandbox/` is a doc and follows D1.

## Open risks

- The peer run `umbrella-anchor` changes `internal/fabricengine/originreconcile.go`, `internal/adoptcli/adopt.go` and their docs and tests, which carry `lyx resume` strings; a textual merge conflict there is possible and resolves by keeping both changes.

## Acceptance criteria

- `lyx attach`, `lyx pause`, `lyx resume` and `lyx start` are not root commands; `lyx a`, `lyx p`, `lyx r` and `lyx prime` run what they ran before.
- `lyx loom start` keeps its flags, including the hidden `--no-attach`.
- The shortcut tests pin the shipped set `a`, `p`, `prime`, `r`.
- No Go source, doc or stencil under the repository names `lyx attach`, `lyx pause`, `lyx resume` or `lyx start` as a command.
- Refusals that named `lyx resume` name `lyx tender resume`, and their existing tests assert the new text.
