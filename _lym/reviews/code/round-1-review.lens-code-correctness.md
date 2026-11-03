```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness — I ran each function and branch the diff adds or changes on concrete inputs, including failure cases.

**Per-function list** (case run, then verdict)

- `Row.SteerFor` / `Row.NoticeFor` (`rows.go:52-65`)
  - Cases: a nil `EngineTexts` map, an engine with no entry, and an entry with one empty field.
  - Each returns the plain text.
  - Correct.
- `codexSteerPythonDeny`, `codexSteerShellEditDeny`, `codexSteerHeadlessClaudeDeny` and their notices (`rows.go:86-102, 189`)
  - Cases: forbidden characters, a doubled full stop, and the notice-once count.
  - There are no quotes or backslashes.
  - The shell-edit and headless notices are the steer plus `.`, so there is no double stop.
  - Only the shell-edit notice carries the Codex edit rule, so it is stated once.
  - Correct.
- `checkTexts` (`rows.go:416-432`)
  - Cases: a planted bad character in an engine steer and in an engine notice, and an engine with only a steer.
  - Both plants are refused naming the row.
  - An empty field passes.
  - Correct.
- `installedGuards` (`guards.go:37-49`)
  - Cases: every switch on, all off, and no refuse list.
  - It returns the table order and drops only the switch-off rows.
  - An unknown switch panics.
  - Correct.
- `guardCommand` (`guards.go:61-68`)
  - Cases: a one-pattern row, a two-pattern row (computed-rm), and a pattern holding `'`.
  - Each is `in=$(cat); {grep}&&{grep}&&echo 'deny'||printf '{}\n'`, and every path exits 0.
  - A grep miss falls through to the empty output, and `echo` succeeds on a match.
  - Patterns are posix-quoted.
  - The steer has no `\` or `$`, so `echo` expands nothing.
  - `TestGuardCommand_DeniesAMatchingCommand` runs this under sh.
  - Correct.
  - Non-finding: a row with zero patterns would render `in=$(cat);  && echo …`, a syntax error.
  - No such row exists, and that would be internal misuse.
- `denyDecision`, `denyNotice` (`guards.go:54, 80`)
  - Cases: no rows, and several rows.
  - No rows gives `""`, and several are space-joined in order.
  - Correct.
- `hookSettings`, `hookEntry`, `hookSetting` (`command.go:122-145`)
  - Cases: no guards, and guards.
  - With no guards there is no PreToolUse setting.
  - With guards, one setting holds every entry, sitting between the recorded hooks and the activity hook.
  - Correct.
- `Prepare` (`codexengine.go:79-152`)
  - Cases:
    - A gate config that fails to load refuses the launch before anything is written.
    - An absent gate config installs the rows with no refuse-raw row.
    - `DenyPublish` installs the publish row.
    - A spec with skills where none resolves.
  - The last one sends `firstTurn` as "No skill file is to be read now…" with the pointer as `PromptLine`. Shuttle then loads, finds all unknown, and sends the pointer. Consistent.
  - Correct.
- `promptText` (`codexengine.go:155-161`)
  - Cases: an empty notice, and a non-empty one.
  - An empty notice is dropped, and a non-empty one comes first.
  - Correct.
- `skillLoadLine`, `SkillLoadMessage` (`skillload.go:39, 73`)
  - Cases: none found, and some found with one absent.
  - The result is a single line, absent skills are left out, and none found gives the no-op line.
  - Classification rebuilds the same line, so the load message matches in the rollout.
  - Correct.
- `skillPath` / `skillVersion` (`skillload.go:116-136`)
  - Cases: `1.9.0` against `1.10.0`, two roots, `"prose"` with no plugin, and a missing root.
  - The newest version by number wins under the first root that holds it.
  - A missing root gives an empty `Glob` result and `""`.
  - Correct.
- `compareVersions` (`skillload.go:140-154`)
  - Cases: a prerelease, a git-SHA or odd directory name, and mixed numeric and text fields.
  - See finding 2.
- `ClassifySkillLoad` (`skillload.go:47-70`)
  - Cases: no or empty `transcript_path`, an absent file, no matching user message, a read before the message, a read after it, and an absent skill.
  - An empty path, an absent file and no matching message all give an unverified report.
  - A read before the message gives missing, a read after gives loaded, and an absent skill gives unknown.
  - Correct apart from finding 1.
- `loadTurnCalls` (`skillload.go:175-208`)
  - Cases: a truncated last line, a line with no payload, a non-object payload, and the same message arriving as both a `response_item` and a `user_message` event.
  - Each bad line is skipped.
  - Either message form resets the calls.
  - A user message that differs from the load message falls through without resetting.
  - Correct.
- `messageText` (`skillload.go:211-225`)
  - Cases: string content, and a list of blocks.
  - String content gives `""` (unverified, safe), and a list of blocks is joined.
  - Correct.
- `skillRoots` (`skillload.go:84-100`)
  - Cases: a config that fails to load, and a root that does not expand.
  - A config that fails to load gives no roots plus a Warn, and a bad root is skipped with a Warn.
  - Correct.

**Findings**

1. LOW, `design`, `skillload.go:63` and `skillload.go:156-160`.
   - The loaded check is a bare substring match on `skills/<name>/SKILL.md` across the whole raw call payload.
   - A read of another plugin's same-named skill, such as `other:prose`, marks `scribe:prose` loaded.
   - A command that only names the tail also counts.
   - D9 prescribes the tail match and accepts "a read the tail match misses reads missing", but not the false-loaded direction.
   - Evidence: `skillload.go:63`; the test rows in `skillload_test.go:115-129` do not cover it.
2. LOW, `design`, `skillload.go:140-154`.
   - Version ordering is not a total order for non-numeric or mixed fields.
   - Numeric fields compare as numbers and the rest as text, so `2 < 10 < 1a < 2` forms a cycle, and `slices.MaxFunc` then depends on glob order.
   - A prerelease field `0-beta` text-compares greater than `0`, so `1.0.0-beta` is picked over `1.0.0`.
   - A hash-named version directory orders arbitrarily against `1.1.0`.
   - It is deterministic only because `Glob` returns sorted results.
   - Evidence: `skillload.go:145-148`; no test covers a non-numeric directory.
3. LOW, `design`, `skillload.go:122-125`.
   - `filepath.Glob` returns `ErrBadPattern` when a configured root holds a glob metacharacter such as `[`.
   - The code treats that as "no root holds it" and reports the skill unknown with no Warn.
   - A skill is then dropped silently, whereas a missing root is correctly silent.
   - Evidence: `skillload.go:122-125`.
4. LOW, `design`, `skillload.go:53` and `skillload.go:84-100`.
   - `ClassifySkillLoad` re-resolves the roots and versions at classification time and rebuilds the expected load line from the files then present.
   - If the plugin cache gains a newer version between launch and the turn end, the rebuilt line no longer equals the typed one.
   - The report then degrades to unverified instead of loaded or missing.
   - It is safe and fails soft, but it silently skips verification and the retry.
   - Evidence: `skillload.go:53`.

The rest of the diff is correct on the cases above.

**Tool calls:** none (0 of 6). My context already held every file and caller I needed.
