---
round: 2
focus:
  - "internal/selfreportcli/cli.go, Command and resolveBody: does every use of the body and body-file flag names go through one named constant each, with no raw literal left, and does the shorthand and help text still read correctly?"
  - "internal/selfreportcli/cli.go, existingFileNamedBy and resolveBody: does a -b text that names an existing regular file, with or without surrounding whitespace or a trailing newline, get refused with a way forward that quotes the path, while a directory name or an empty string is filed as text?"
  - "internal/selfreportcli/cli_test.go, TestCommand_RefusedBeforeTransport and TestRunCreate_Success: do the added rows assert behavior no other row covers, and do the wantOutput strings match the refusal's current format?"
---

The focus follows the round's gating findings and the text any fix to them adds.
