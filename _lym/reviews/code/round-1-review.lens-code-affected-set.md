```yaml
lens: code-affected-set
outcome: reported
```

**Per change, the set I found and whether the diff covers it**

- **New `ask` and `ask_default` keys on `buttonEntry` and `buttonSpecEntry`.**
  - Covered: `buttons.go`, `buttonrow.go`, both templates' `buttons:` comments, the `reedengine` click-flow paragraphs in `doc.go`, and `config_test.go`.
  - Not covered: two sentences of `internal/reedengine/doc.go` still describe the old shape (finding 1).
  - Not an issue: `desktophook/template.yaml:4` and `desktophook/config.go:31` point at "shaped as reed.yaml's buttons" and name no fields.
- **New `--answered` and `--answer` flags, and the `validateButtonFlags` case.**
  - Covered: `reedcli/button.go`, `reedcli/doc.go`, `button_test.go`.
  - Not covered: the `internal/reedcli/cli_test.go:75` doc comment names only `--confirmed` (finding 3).
- **`composeConfirmedButtonLine` renamed to `composeButtonReentryLine`, and `fillConfirmPlaceholders` to `fillPromptPlaceholders`.**
  - Covered: no reference to either old name remains anywhere. `panebin_test.go` has no hit for the old name in my grep either.
- **`tmuxQuote` split into `tmuxString` and `tmuxQuote`.**
  - Covered: both tests pin it.
- **The `buttonClickIntegration` real-tmux test.**
  - Not covered: it has no `ask` entry. Its comment and entries (`internal/reedengine/buttonclick_integration_test.go:20,46`) cover only the confirm path. The `command-prompt` template and `%%%` escaping are therefore never run against a real tmux, and tmux's actual escaping is what the whole design rests on (finding 2).
- **Shipped `agent` and `term` button entries.**
  - Covered: both templates, `config_test.go`, and the `TemplateTaskGroupComesBackWhole` row in `buttons_test.go`.
  - The old `term` label survives nowhere else: grep over `.go`, `.yaml` and `.md` found no other reference.
- **New session option `@lym_button_answer`.**
  - Covered: `doc.go`'s decision-bearing option list.
  - Not affected: `bar.go` and `follow.go` mention the other button options, not this one.
- **PATTERN entries and their background files.**
  - `PATTERN-pane-text-injection`: not touched. Text reaches a tmux prompt, not an agent pane, so the entry doesn't bind this change.
  - `PATTERN-test-speed`: the new `TestValidateButtonFlags` opens with `tiergate.Require(t, 0)` and `t.Parallel`, as the entry requires.
  - `PATTERN-refusal-way-forward`: binds `webster*`, `shed*` and `loom*` only, not `reedengine`.
  - No `Pattern-Departure:` trailer is present, and none is needed.
  - The directive declares no PATTERN changes, so none are owed.

**Findings**

1. **LOW, `consistency`: `reedengine/doc.go:67` and `:72` still describe the pre-change entry.**
   - Line 67 says the validity rules cover "no '#', '%' or control character in a label or confirm text". The diff also applies that rule to the ask text, and adds a comma ban on `ask`, `ask_default` only with `ask`, and `{answer}` only with `ask`.
   - Line 72 describes the spec entry as "its label, its run and confirm text … and whether {strand} is left for the click". It omits the new `ask` and `ask_default` fields, and `{strand}` can now also sit in the ask text.
   - The diff edited this file's nearby click-flow paragraphs (lines 87–96, 107) but left both sentences untouched.

2. **LOW, `consistency`: no real-tmux coverage of the ask path.**
   - `buttonclick_integration_test.go:20,46` was left as is.
   - The `command-prompt` argv, its `%%%` template and the answer round-trip through `@lym_button_answer` get no real-tmux coverage. The unit rows use a fake tmux that only compares argv strings, which cannot catch a wrong template.

3. **NIT, `consistency`: stale summary in `reedcli/cli_test.go:75`.** It lists only a `--confirmed` without `--hash` among the refusals the button flags test covers. The diff added the matching `--answered` refusal, which `TestValidateButtonFlags` pins instead.

**Tool calls**
1. `grep` for button docs and templates outside the changed files. My context didn't hold the other docs and templates describing button entries.
2. `grep -l` for other users of the button verb's flags and options. The manifest named no consumers beyond the files already read.
3. `grep` and `sed` on the integration test, `desktophook/template.yaml`, `cli_test.go`, `stencilseedgate_test.go` and `doc.go` lines 64–76. I hadn't read any of them.
