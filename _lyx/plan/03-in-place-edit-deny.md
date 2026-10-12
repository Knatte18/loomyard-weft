# Card 3 — in-place-edit-deny

**Create:**
- `plan:internal/shuttleengine/claudeengine#steerInPlaceEditDeny` -> `const steerInPlaceEditDeny = "scripted in-place edits are refused here under the edit directive in your prompt; reading with sed -n is fine"`
- `plan:internal/shuttleengine/claudeengine#noticeInPlaceEditDeny` -> `const noticeInPlaceEditDeny = steerInPlaceEditDeny + "."`
- `plan:internal/shuttleengine/claudeengine#inPlaceEditCommandPattern` -> `const inPlaceEditCommandPattern = commandPositionPattern`
- `plan:internal/shuttleengine/claudeengine#TestInPlaceEditDenyHook_DeniesInPlaceScriptEditsOnly` -> `func TestInPlaceEditDenyHook_DeniesInPlaceScriptEditsOnly(t *testing.T)`

**Edit:**
- `internal/shuttleengine/claudeengine#standingDenies`
- `internal/shuttleengine/claudeengine#steerPythonDeny`
- `internal/shuttleengine/claudeengine#TestBuildSettings_DenyToggleMatrix`
- `internal/shuttleengine/claudeengine#TestBuildSettings_AgentAndBashHooks`
- `internal/shuttleengine/claudeengine#TestBuildDenyNotice_SentenceContent`
- `internal/shuttleengine/claudeengine/doc.go`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#commandPositionPattern`

**Intent:**
D2 and D4: a standing PreToolUse(Bash) deny refuses an in-place script edit, and the python deny's steer names `sed -n` among the readers.

`inPlaceEditCommandPattern` is the grep -E pattern over the raw PreToolUse payload JSON, built on `commandPositionPattern` like `pythonCommandPattern`, so the head's value is only its start.
It matches `sed` or `perl` in command position, optionally path-prefixed, as a whole word followed by whitespace, then that command's words up to the next `;`, `&`, `|`, `)`, backtick or JSON-escaped newline, then whitespace and an in-place flag word:
- for `sed`, `-i` or `-i<suffix>`, a short-flag cluster holding `i` such as `-ni` or `-Ei`, or a `--` word that is a prefix of `--in-place` at least as long as `--i`, optionally followed by `=<suffix>` and ending at a word end;
- for `perl`, a short-flag cluster holding `i`, such as `-i`, `-i.bak` or `-pi`.
A short-flag word opens after whitespace with a single dash followed by letters, so `--quiet`, `--silent`, `--expression=` and `--file=`, which hold an `i`, never match as a cluster.
GNU sed accepts any unambiguous abbreviation of a long option and no other sed long option starts with `--i`, so every prefix of `--in-place` from `--i` up is denied and no read is.
The pattern holds no single quote, since the hook command wraps it in one.
Its doc comment states the bound below.

The new `standingDenies` row, after the raw go row, matches `Bash`, runs `grep -Eq '<pattern>' && echo '<deny JSON>'; true` like the python row, carries `steerInPlaceEditDeny` and `noticeInPlaceEditDeny`, and is installed in every run, the orch included, with no config key.
The steer points at the edit directive's rule and does not restate it, and holds no quote or backslash, as the init check requires.
Its row comment states the bound: a guardrail, not a barrier.
It lets through `sed` writing by redirection (`sed ... > f`), a `w` command inside a script, `awk`, `find -exec sed -i`, anything behind `bash -c`, `eval` or a variable, and anything after a prefix `commandPositionPattern` does not list (`nohup`, `command`, `timeout`).
It falsely denies a matching spelling in a quoted argument, in a `grep` alternation such as `x\|sed -i`, or in another payload field, as the python row does, and a `perl` read whose cluster holds an `i`, such as `perl -Mstrict -ne`.

`steerPythonDeny` becomes "lyx agents never run Python; edit files with Edit or Write, search and read with grep, awk, cat or sed -n", still free of quotes and backslashes; `noticePythonDeny` follows it.

`TestInPlaceEditDenyHook_DeniesInPlaceScriptEditsOnly`, in `hookcmd_integration_test.go` beside the python row's test and in its table shape through `denyHookCommand` and `preToolUseOutput`, pins the deny JSON for `sed -i 's/a/b/' f`, `sed -i.bak s/a/b/ f`, `sed -ni 3p f`, `sed --in-place s/a/b/ f`, `sed --in-place=.bak s/a/b/ f`, `sed --in s/a/b/ f`, `sed --in-pl=.bak s/a/b/ f`, `sed -n 3p f -i`, `perl -pi -e s/a/b/ f`, `/usr/bin/sed -i s/a/b/ f`, and `sed -i s/a/b/ f` after `cd x && `, after `true; ` and after `cat f | `.
It pins empty output and exit 0 for `sed -n 1,5p f`, `sed s/a/b/ f`, `sed --quiet 3p f`, `sed --silent 3p f`, `sed -n 3p f` then a newline and `grep -i x f`, `grep -i x f` and `git diff`.
The file's header comment names the new deny beside the python one.

`TestBuildSettings_DenyToggleMatrix`'s count check for a run with every toggle off now expects the in-place deny beside the raw go deny, and the in-place deny is asserted installed under every input, as the raw go deny is.
`TestBuildSettings_AgentAndBashHooks` sets the in-place row's commands apart from the fork guard's as it does the raw go row's, and asserts exactly one in every case.
`TestBuildDenyNotice_SentenceContent` asserts `noticeInPlaceEditDeny` is in the notice of an autonomous, an interactive and a fork run.
`TestBuildDenyNotice_MatchesInstalledDenies` needs no change: it already ties every row's notice to its installed hook.

`claudeengine`'s `doc.go` gains a paragraph on the in-place row beside the python and raw go ones: what it denies, that every run installs it, and its bound.
`docs/overview.md`'s shuttle row names the in-place deny beside the python and raw go guardrails, and says the python deny's steer names `sed -n` as a reader.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/shuttleengine/claudeengine` by one for the new test.

**ImpactSummary:** Adds one always-installed Bash PreToolUse hook and one notice sentence to every spawned session, the orch included, and rewords the python steer; the settings and notice tests and the claudeengine budget row follow.
