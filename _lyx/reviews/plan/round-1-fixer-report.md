# Fixer report — plan, round 1

Every finding in `round-1-review.md` is fixed.
`lyx loom validate-plan` passes after the edits (`ok: true`, informational `create-new-unit` lines only).
`approved:` in `00-overview.md` is untouched.

## Changes

- **F1** (`02-config-load-fills.md`): added `internal/websterengine/config_test.go` to the Edit list, with a test bullet.
  The two absent-key cases of `TestLoadConfig_MissingNumericKnobNamesTheKey` become positive cases; the explicit-zero case keeps its refusal.
  The ImpactSummary now names the four packages whose refusal tests flip, and notes that webster's positive-integer check still guards an explicit zero.
- **F2** (`01-yaml-fill-missing.md`, `02-config-load-fills.md`): card 1's test list gains parse-error cases for an unparseable template and an unparseable `existing`.
  Card 2's configengine cases gain an unparseable present file refusing with its path named.
- **F3** (`02-config-load-fills.md`): `TestLoadOrTemplate_PresentCommentsOnly` joins the flipped configengine tests.
- **F4** (`06-ide-spawn-clean.md`): the tracked-`tasks.json` integration case runs on a task pair and on the prime.
  On the prime it asserts the hub workspace file is still written and launched.
- **F5** (`06-ide-spawn-clean.md`): the `spawn_test.go` deletion now moves its prime-resolution degrade case into the new integration file, with a matching test bullet: a non-git `Location` spawning a real pair logs the warning and launches the bare folder.
- **F6** (`05-reed-add-unless-flag.md`): the Intent and help paragraph now state the flag's precedence and independence.
  `--unless-name` stands alone, without `--name` or `--if-absent`.
  It is evaluated before `--if-absent`'s match, so a live match also suppresses `--if-absent`'s relaunch.
- **F7** (`02-config-load-fills.md`): added `internal/loomcli/wiring_test.go` and `internal/webstercli/wiring_test.go` to the Edit list, with a bullet rewording their "strict on missing keys" doc comments.
- **F8** (`02-config-load-fills.md`): step 3 now describes the backstop's full reach: element keys of a list of mappings that the file's list lacks, whether the element is present or the list is shorter.

## Deferred

None.
