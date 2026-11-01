# Card 3 — button-entries

**Create:**
- `internal/reedengine#ButtonPlace`
- `internal/reedengine#ButtonPlaceTask`
- `internal/reedengine#ButtonPlaceBeam`
- `internal/reedengine#ButtonPlacePrime`
- `internal/reedengine#buttonEntry`
- `internal/reedengine#placeButtons`
- `internal/reedengine#buttonEntryProblem`
- `internal/reedengine#fillRunPlaceholders`
- `internal/reedengine#fillConfirmPlaceholders`
- `internal/reedengine#TestPlaceButtons`
- `internal/reedengine#TestFillButtonPlaceholders`

**Extends:**
- `internal/reedengine/doc.go`

**Uses:**
- `internal/reedengine#Config`
- `internal/shell#Shell`

**Intent:**
A new file `internal/reedengine/buttons.go` holds the decoding, validation and placeholder filling of button entries (D2, D6), as pure functions the render (card 6) and the click verb (card 8) call.
Nothing calls them yet outside the tests; reed still renders no row.

- `ButtonPlace` names a session's place with the three constants; card 4 tells it through the geometry.
- `buttonEntry` holds `Label`, `Run`, `Confirm` and `Window`, each decoded from the entry key of the same name in lower case, and the entry's index within its group.
- `placeButtons(groups, place, source)` returns the valid entries of the group `place` selects in `groups`, the `buttons` node of one config file, in config order.
  For `ButtonPlacePrime` it takes the `prime` group when that group decodes to a non-empty list, else the `beam` group (D2, D3); a `prime` group that is not a sequence is skipped with the invalid-group Warn below and the `beam` group is taken.
  A zero `yaml.Node` (no `buttons` key, as a hand-built `Config` has) or an absent group gives no entries and no log.
  A group that is not a sequence, a mapping included, and an entry that does not decode into `buttonEntry`'s string fields (a scalar entry, or a nested list as a `label`) are skipped with one `logger.Warn` naming `source`, the group and, for an entry, its index; so is every entry `buttonEntryProblem` refuses, the Warn naming the problem too.
  The other entries are still returned, so a bad button costs only itself.
- `buttonEntryProblem` returns the empty string for a valid entry, else one sentence naming the first problem, in this order: an empty `label`, an empty `run`, a `#`, a `%` or a control character (`unicode.IsControl`) in `label` or in `confirm`, a `window` other than empty or `agents`, and `{strand}` in `run` or `confirm` of an entry without `window: agents`.
- `fillRunPlaceholders(sh, run, values)` replaces each `{name}` whose name is a key of `values` with `sh.Quote(value)`, so a value holding spaces stays one word; brace text naming no key of `values`, such as an awk program or `{strand}` before the click, stays literal.
- `fillConfirmPlaceholders(confirm, values)` replaces the same placeholders with the plain value after doubling each `#` and each `%` in it, so a value cannot inject a status sequence or a strftime field into the `confirm-before` prompt.

`source` is the file label the render passes.
The render reads reed's node after the layers are stacked, so no single layer is known; its label names both files that may hold the entry, `reed.yaml (the hub base or the worktree file)`, and the machine's label is `desktop.machine.yaml`, the one layer that adds machine buttons (D2's Warn naming the file).
`internal/reedengine/doc.go` gains a sentence naming `buttons.go` as the one decoder of button entries and the validity rules it applies.

Tests, in a new test file under `//go:build tier0`, internal package:
- `TestPlaceButtons` is one table over YAML documents decoded into a node, reading the logs through `logcapture.CaptureVerbose`; since that capture repoints the logger's process-global output, the test does not call `t.Parallel`, and a comment at the test names that state (`PATTERN-test-speed`).
  Rows: the template's `task` group comes back whole; `prime` with an empty `prime` gives the `beam` entries and with a non-empty `prime` gives only `prime`'s; a `prime` group written as a mapping gives the `beam` entries and a Warn naming `prime`; a zero node and a node lacking the selected group give none and log nothing; a group written as a mapping, a scalar entry, an entry whose `label` is a list, each invalid entry of `buttonEntryProblem`'s list (empty label, empty run, `#` in label, `#` in confirm, `%` in label, `%` in confirm, a tab in label, a tab in confirm, `window: all`, `{strand}` in `run` without `window: agents`, `{strand}` in `confirm` without `window: agents`) is skipped while its valid neighbour is returned, and the Warn names `source`, the group and the index; an entry with `window: agents` and `{strand}` is kept.
- `TestFillButtonPlaceholders` (`t.Parallel`, no log read) is one table over both dialects: a `{worktree}` value `/a b/c` reaches `run` as `sh.Quote("/a b/c")`; `awk '{print $1}'` and `{strand}` with no `strand` value stay literal; an unknown `{x}` stays literal; a confirm value `a#b%c` reads `a##b%%c`.

**ImpactSummary:** a new file of pure helpers and one doc sentence; no existing code changes and no caller exists until card 6.

**Commit:** 3: Decode, validate and fill reed status-bar button entries
