---
round: 2
focus:
  - "Go comments added or changed by the diff in internal/yamlengine, internal/boardengine, internal/configreg, internal/configcli and internal/selfreportcli: does each sentence sit on one line, with a further break only at an independent-clause boundary, as PATTERN-comment-line-breaks requires? Check the whole changed comment, not just its first line."
  - "The `config` bullet in the module list of docs/overview.md against the CLI behavior of `lyx config` after the picker was removed: does it describe bare `lyx config` as listing and `lyx config <module>` as editing, and agree with the task line near the top of the same file?"
  - "Helper functions added in internal/yamlengine/set.go and reconcile.go: does each carry a doc comment consistent with its siblings in the same file?"
---

The focus entries name where to look; the base diff review still applies in full.
