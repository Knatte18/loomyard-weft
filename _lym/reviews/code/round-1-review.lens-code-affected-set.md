```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each thing the diff changes, I looked for what reads or asserts it outside the lines it touches. I made three tool calls.

**Set per change**

1. **`attribution` key added to `settingsDoc` and `openSettingsDoc`.**
   - Writers and builders: `buildSettings` is called only from `claudeengine.go:208`, and `buildOpenSettings` only from `writeOpenFiles` (`command.go:480`). Both marshal the structs, so no other path composes a Claude settings document. Covered.
   - Tests: `TestBuildSettings_MatchesGolden` with the golden file, `TestSettings_CreditNoSession`, and `checkOpenCreelFiles`. Covered.
   - Other readers: `hookcmd_integration_test.go:32` and the `settings_test.go` callers read only hook entries, so the new key does not affect them. Covered.
2. **Doc comments describing the open-launch settings document.**
   - Updated: `settings.go:1,20,184,197`, `doc.go:141,312`, `command_test.go:531`, and the overview sentence.
   - Not updated: `command.go:404` and `command.go:464`, which are listed under finding 1. **Not covered.**
3. **`docs/overview.md`.** The added sentence sits in the shuttle bullet at line 430, after the `append-system-prompt.txt` and `[creel]` mail text. It names `attribution` and says both documents carry it. Covered.
4. **PATTERN entries.**
   - `PATTERN-shuttle-provider-seam` is satisfied: the change stays under `internal/shuttleengine/claudeengine`.
   - The `PATTERN-stencil-ownership` entry (stencils and specs) is not touched, and no stencil is changed.
   - The directive's declared changes list was not in my context, so I cannot confirm "Declared PATTERN changes landed". The diff stat shows no `PATTERN.md` or `pattern/` change.
5. **Commit messages.**
   - `711aec52` carries no `Pattern-Departure:` trailer and needs none.
   - `16bac6c9` ends with `Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>`. That is an engine credit on this branch's own commit, which the change forbids. This is finding 2.

**Findings**

1. **MEDIUM, `consistency`.** `internal/shuttleengine/claudeengine/command.go:404`: the `OpenLaunch` doc says the written `settings.json` holds "creel's hook delivery alone". It now also holds `attribution`, so the sentence contradicts `doc.go:312` and `settings.go:198`. The doc comment of `writeOpenFiles` at `command.go:464` says only "holding creel's hook delivery over the flag directory". It is incomplete rather than false, so I suggest the fix covers it too. This is a package doc the Wanted list asked to keep in step with the document.
2. **LOW, `consistency`.** Commit `16bac6c9` (subject: "fix: open-launch settings test accepts the attribution key beside hooks") ends with `Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>`. The record's Goal says no engine is credited as a co-author, so a commit of this change credits one. The body of `711aec52` makes no such claim, and the trailer belongs to this branch's own commit. Evidence: `git log` output in my inherited context.
3. **LOW, `consistency`.** `_lym/landing/summary.md` says "the open-launch paragraph no longer says the creel-only document holds hooks alone". The `docs/overview.md` diff is one added sentence, and a search found no "hooks alone" wording in the overview. The claim is borne out only by `doc.go:312`, so its reference to the overview is unbacked. Evidence: the word-diff of `docs/overview.md` and the grep above.

**Tool calls**
1. `grep` over `*.md`, `*.go`, `*.yaml` and `*.tmpl` files for wording describing the open settings document, because my context lacked a repo-wide search for stale phrasing. It found `command.go:404` and `command.go:464`.
2. `grep -o` on `docs/overview.md` for the added sentence's context, because the diff's huge single line hid where it sits.
3. A `git diff` word-diff of `docs/overview.md`, run earlier in the inherited context, which showed the overview change is only the added sentence.
