```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each function and branch the diff adds or changes through concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

1. **Render loop in `TestBuildSettings_MatchesGolden`** (`settings_test.go:1168-1180`). Correct.
   - Case: the masks 0 to 15. `goldenLabel` is injective over the four bools, so `got` holds 16 distinct labels.
   - Failure case: `buildSettings` returns an error. `t.Fatalf` fires before the golden is read or written, so no partial golden is written.
   - This is stricter than at the base, which skipped `buildSettings` for a label the golden lacked.
2. **`-update` branch** (`settings_test.go:1182-1191`). Correct.
   - Case: the current code. `json.MarshalIndent` sorts the map keys, indents by two spaces and HTML-escapes `&lt;`, `&gt;` and `&amp;`. That matches the committed file's `<` and `&` and its sorted labels.
   - With the appended `'\n'`, the write matches the file's tail bytes `}\n`.
   - The relative path resolves against the package directory, which is the test binary's cwd. The gate runs the tier command at the module root (`tierrun.go:526`, `resolveRunDir`), and `go test` then runs each binary in its package directory.
   - Failure case: `WriteFile` fails. The test fails loudly through `t.Fatalf`; a partial write is possible, but it is only a test-time artifact.
   - The early `return` skips the stale-label check. That is right, since the file is now exactly `got`.
3. **Compare branch with a golden that is missing, holds `null` or holds an extra label** (`settings_test.go:1193-1217`). Correct.
   - A missing file is fatal. `null` unmarshals to a nil map, so every label reports "golden lacks", as at the base.
   - An extra label in `want` is still reported through `got[label]`.
   - A settings or notice difference is still reported, with the got/want order unchanged.
   - The only change is that error messages now come out in random map order. That has no correctness effect.
4. **`-update` through `lym gate test`** (`cli.go:108-109` → `tiers.go:137`). Correct for the documented call.
   - The words after `--` reach the tier's `{args}`.
   - The `-run` filter blocks the carry skip (`tiers.go:141`, `doc.go:237`).
   - The green record keys on the args (`green.go:448`), so an update run never stands in for a plain run.
   - The rerun at `tierrun.go:787` forwards `-update`, but it runs only after a failure, which the update branch does not produce.
5. **New test row `system_flag_of_a_later_dbus_send`** (`rows_integration_test.go:316`). Correct.
   - `unquotedWordOtherThan` crosses `/` and `x.y`, then `;` hits `searchEnd`, so the row denies. The expectation `true` is the right direction.
6. **Doc claims on `unquotedWordOtherThan`, checked against its body and `sessionBusCommandPattern`** (`rows.go:417-448`). The word-level claims hold; the fail-closed claim does not (finding 1).
   - `--system` exactly blocks crossing in every case I ran: followed by a space, by `;` or by the payload's end.
   - These words are crossed: `--syst`, `-`, `--systemd` and `--system=1`. A word with a quote, such as `--sys"x"`, matches at the quote, so the command is denied.
   - The single-quoted and double-quoted wrapped-shell scopes behave the same way, because the word unit itself excludes `'` and `\`.
   - At a stop, the session-bus row matches only because its own `searchEnd` (`rows.go:420`) holds every character the word unit excludes (`rows.go:436`), whitespace aside. The helper does not supply that end.

**Findings**

1. MEDIUM, design, `internal/shellguard/rows.go:428-431` and `internal/shellguard/doc.go:36-38`. The docs promise a fail-closed guarantee that the helper does not give, and the end set they name is incomplete. Following them as written gives a fail-open exception.
   - The helper's doc says "The run stops at anything it cannot read and matches there, so the guard fails closed". The helper returns only the word pattern, so the match at a stop comes from the caller's end alternation.
   - Both docs name that end as "a separator, a quote or the payload's end". A row author who writes only that end, for example `([;&amp;|"']|$)`, leaves out the backslash, backtick, `&lt;` and `&gt;`.
   - Example: a payload `foo \"x\"` without the flag. In the JSON a quote is `\"`, so it opens with a backslash. The run stops at `\`, nothing matches there, and the refused command passes. `foo &gt; out` passes the same way at `&gt;`, and so does a `\t` or `\n` escape.
   - Fail-closed needs the end to cover every character the unit excludes. That coupling is kept only by hand: the unit's class at `rows.go:436` and `searchEnd` at `rows.go:420` are written separately.
   - Fix: have the docs state that the run must be ended by an alternation of every character the word unit excludes, plus `$`. Better, have the helper return that end itself.

Tool calls:
- `grep request.args` in `internal/gatecli`: I needed to know whether a rerun, the carry or the green record forward `-update`, and my context had only the arg split at `cli.go:108`.
