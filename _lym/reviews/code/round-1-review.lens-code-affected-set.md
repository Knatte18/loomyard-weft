```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. Which consumers, tests, docs and rule documents the refuse-raw notice change touches, and whether the diff leaves any of them wrong.

**Per change, set found, covered or not**

1. **`refuse-raw` `Row.Notice` text shortened.**
   - **Renderer:** `internal/shuttleengine/claudeengine/settings.go:317` copies `row.Notice` into `standingDeny.notice`. `buildDenyNotice` (`settings.go:323-328`) joins every installed notice, and `appendPromptText` writes them into the append-system-prompt file. No other package renders `Row.Notice`. Covered.
   - **Raw-go in the same prompt:** `raw-go` uses `SwitchNone` and is built unconditionally in `rowsBeforeRefuseRaw`. The refuse-raw row exists only when the raw-go row does, and `raw-go` precedes it. So the gate rule appears once in a prompt with a refuse-raw notice, and the notice is never emitted without it. I did not check that `shellGuardDenies` installs every row with `SwitchNone`. Covered.
   - **Consumer tests:** `settings_test.go:896` asserts the prompt contains `refuseRow.Notice` by reading the row, so it follows the new text. `settings_test.go:448` compares `deny.notice` to `row.Notice`, and `settings_test.go:1027` compares `buildDenyNotice` to a join of the same notices. Both are text-agnostic. Covered.
   - **Gate-rule wording assertion:** `settings_test.go:974` checks that the non-fork notice contains "Gate rule: …3600000…" and "never cd into…". Both strings come from the raw-go notice, which is unchanged, so it still passes. Covered.
   - **Steer consumers:** `norecoverydeny_test.go` and the hook command (`settings.go`) use Steer and Forms. The refuse-raw steer text is unchanged, apart from being built from `refused + ". Gate rule…"`, which produces the same string. Covered.
   - **Character check:** `checkTexts` and the `claudeengine` init check forbid quotes, backslashes and newlines in notices. The new notice has none, and `;` is allowed. Covered.

2. **`Rows` doc comment reworded** (`rows.go:157-158`). It is accurate: the gate rule appears in the raw-go steer and notice and in the refuse-raw steer. The matching comment at `settings.go:273` ("which the raw go row's steer and notice carry") still holds, because that variable is the foreground phrase for the raw-go row. Covered.

3. **New paragraph in `shellguard/doc.go`.** It matches `rows.go`: the rule is stated once in the raw-go notice, the refuse-raw row's notice names only what it refuses, and each steer carries the full rule. The raw-go steer's rule is the `--tier` flag form, not the `--card` form, which is only a detail. Covered.

4. **Docs and rule documents.**
   - **`claudeengine/doc.go:247-249`:** it describes the refuse-raw row as denying prefixes in `refuse_raw`, with the row rendered into settings and notice. It does not quote the notice text. Covered.
   - **`gateslot/doc.go`, `gatecli/doc.go`, `docs/dotnet-hub.md`:** they mention `refuse_raw` only as a config key. Covered.
   - **`PATTERN-shell-guard-table`:** its rule is "only `internal/shellguard`'s table, every engine rendering every row". The notice is still rendered for every row, so the change does not conflict. No `pattern/` background file exists for it, and no declared PATTERN changes apply. Covered.
   - **Stencils:** nothing in the diff touches one, and no stencil quotes the notice. Covered.

5. **Commit message.** It makes no pattern-departure claim and needs no trailer. Its claims match the diff. Covered.

**Findings:** none from this lens.

**Tool calls (3):**
1. `Bash`: read the `Notice` consumers in `cmd/lym/norecoverydeny_test.go` and listed files naming refuse-raw or `refuse_raw`, plus `settings.go:268-276`. My context did not show `norecoverydeny_test.go` or the other doc hits.
2. `Bash`: grepped the docs that name `refuse-raw` or `refuse_raw` for any quoted notice text. I needed to know whether any doc quotes the notice.
3. A `Bash` call was not needed beyond these two. I read the `settings.go` and `settings_test.go` consumer lines from the earlier manifest output.
