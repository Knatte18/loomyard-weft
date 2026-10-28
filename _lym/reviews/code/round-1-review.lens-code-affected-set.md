```yaml
lens: code-affected-set
outcome: reported
```

Task: for each change in the diff, list the set it forces outside the touched lines, and report anything left stale.

**Per change, the set found**

- **Appended-prompt content (`settings.go` `appendPromptText`, new `batchingSentence`):**
  - Covered: `claudeengine.go:226`, `command.go:338`, `doc.go:210`, `docs/overview.md:413`, `prepare_test.go:3` and `:325`, and `settings_test.go:750` all describe or assert the new content.
  - Not covered: the comments at `prepare_test.go:293` and `:295`, and the comment at `settings.go:403` (findings F1 and F2).
  - Unaffected: `command_test.go` only pins the flag, not the content, and the `settings_test.go:850` test checks the refuse-raw notice with `Contains`.
- **init line-break check:** covered; the map loop checks both sentences.
- **Resume and launch lines:** no code change; both already name the file.
- **Eight stencils:**
  - Covered: no other copy of any of the eight is in the repo (the search for the changed lines matched only `drift-template-verify.md` and `burlerengine/template_test.go`).
    - `drift-template-verify.md` is a different stencil and out of scope.
    - `burlerengine/template_test.go` matches only the heading text.
  - Marker positions: `{{.tool_use_rules}}{{.fork_reading}}` is unchanged in `burler-step-1-explore.md`, and the new sentence follows it.
  - Wording: all eight use "message", not "turn".
  - Rules kept: the darn stencil keeps "each leaving the tree green" and "never a raw test command or a module-wide run". The Webster stencil keeps the per-card gate in step 3 and the commit in step 4.
- **Rule documents:**
  - `PATTERN.md` and `pattern/` need no change, and none is declared.
  - `PATTERN-stencil-ownership` holds: the sentences live in stencils, and the Go constant sits in the engine seam.
  - The `CLAUDE.md` docs rule is met: `doc.go` and `docs/overview.md` land in the same commit.
  - The commit message has no `Pattern-Departure:` trailer and none is needed.
  - The commit trailer reads `Co-Authored-By: Claude Opus 5.5`, where the session reminder asks for `Claude Sonnet 5.5`. This is not a repo rule, so no finding.

**Findings**

- **F1**
  - Severity: LOW
  - Class: consistency
  - Location: `internal/shuttleengine/claudeengine/prepare_test.go:293`
  - Summary: the `TestPrepare_AppendSystemPromptMatchesDenyNotice` doc comment still says the file holds "the deny notice, a space and the creel trust sentence, or the sentence alone", but the test now asserts the notice, trust sentence and batching sentence.
  - Evidence: `prepare_test.go:293`, against the changed assertion at `:325-327`.
- **F2**
  - Severity: LOW
  - Class: consistency
  - Location: `internal/shuttleengine/claudeengine/prepare_test.go:295`
  - Summary: the `//testtiming:keep` reason says the test pins the file "equal to the notice and the trust sentence"; it now also pins the batching sentence.
  - Evidence: `prepare_test.go:295`.
- **F3**
  - Severity: NIT
  - Class: consistency
  - Location: `internal/shuttleengine/claudeengine/settings.go:403`
  - Summary: the `noticeTextForbiddenChars` comment says a deny notice shares the file's line "with the creel trust sentence" only; the line now also holds the batching sentence.
  - Evidence: `settings.go:403`.

**Tool calls**
1. One `grep` over the repo for `append-system-prompt`, "trust sentence" and "creel sentence", plus the stencil-line search. I needed it for the other readers of the file and any other copy of the changed stencils, and my context lacked both.
