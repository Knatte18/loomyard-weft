```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that act on the same state, and then the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1 × D2 (board-data.js loaded by board.html through a script tag): held.
- D1 × D4 (section list, project parent handles, split labels, file names, render time and owner): held.
- D1 × D1's acceptance criterion (HTML escaping of `&lt;`, `&gt;` and `&amp;`): broken. The D1 text says `&lt;` is written as a literal `&lt;`, which contradicts the criterion that no literal `&lt;b&gt;` remains.
- D1 × D7 (README section names): held. D7 changes only the markdown wrapping; the section names D1 reads stay the same.
- D2 × D3 (board.html as a fixed output under mixed binaries): held. D2 scopes its claim to one lym version, and D3's bound covers the flip-flop.
- D3 × D5 (a downgrade removes board.html, then `board open` refuses and names `lym board rerender`): held. The file returns until the next write by the older binary, which D3's bound covers.
- D4 × D5 (both name `lym board rerender` as the way forward): held.
- D5 × D6 (the task's cwd is the folder anchor and D5 resolves with ResolveWithin): held.
- D5 × D6 (`"reveal": "silent"` depends on D5 failing with exit status 1): broken. D6 says D5's refusal and a failed opener exit with status 1, but D5 never states an exit status.
- D5 bound × D6 (xdg-open blocks while the task stays active and a second key press gets the running-task prompt): held.
- D6 hook × beam generated mark × reserved keys (splice after `settings`, mark prepended): held.
- D6 `tasks.json` × D6 workspace `tasks` key (`beam start` only in the workspace file, `board open` only in folders): held.
- D7 × the parent's "phone goal is met" entry: broken.
- D7 × the arbiter's "per-body folds are moot" entry: superseded. The arbiter entry left D7 standing "until the parent answers", and the parent has now answered.
- Open risks line 1 × the parent's "phone goal is met" entry: broken. The parent entry says the line is dropped, and it is still there.
- Open risks line 2 × the parent's "open chain and LY window title stay out" entry: broken. The line is settled but not dropped.
- The arbiter's no-launch.json entry × Scope Out: held.
- Constraints × D5 and D6 (fabric boundary, code conventions, test-only code, alt+shift+1 to 6 avoided, no JS runtime): held.
- Acceptance criteria × D1 to D6: held, apart from the D1 escape text above.

**Findings**

1. **BLOCKING · consistency** — D7 together with the Goal, the Scope In bullet "Collapsible Tasks, Notes and Done sections", the D7 acceptance criterion and the D7 auto-mode assumption.
   - Summary: the parent's entry says D7's folds stay only if another goal of the entry needs them. The entry has no goal besides the phone that does, yet the main record still tells Plan to build D7 and keeps the phone half of the Goal.
   - Evidence: `_lym/discussion/decision-record.md:194-198` against lines 6, 18, 170-180, 225 and 260. The board entry body has only the phone and viewer goals.
   - Fix: drop D7, its acceptance criterion and its assumption, the Scope In bullet and the phone half of the Goal.

2. **MEDIUM · consistency** — Open risks, first bullet (the phone part).
   - Summary: the parent's entry says the entry's phone line is dropped from Open risks, but the bullet remains and still asks the parent to confirm.
   - Evidence: `_lym/discussion/decision-record.md:196` against lines 229-232.
   - Fix: drop the bullet.

3. **MEDIUM · consistency** — Open risks, second bullet (`Was ide-prime-setup`).
   - Summary: the parent's "open chain and LY window title stay out" entry settles this line. Under the rubric, a line such an answer settles is dropped.
   - Evidence: `_lym/discussion/decision-record.md:200-204` against lines 233-235.
   - Fix: drop the bullet.

4. **LOW · consistency** — D5 against D6's first bullet.
   - Summary: D6's claim that a failure brings the terminal forward rests on D5's refusal and a failed opener exiting with status 1, but D5 states no exit status for either.
   - Evidence: `_lym/discussion/decision-record.md:131-133` against lines 112-118. The board pre-run aborts with `clihelp.Abort(ctx, 1)` (`internal/boardcli/cli.go` PersistentPreRunE), but how a RunE refusal maps to an exit code is not shown in `cmd/lym/main.go:56-63`, where it comes from `run`.
   - Fix: have D5 state that its refusal and opener failure exit with status 1.

5. **LOW · consistency** — D1, line 43.
   - Summary: the escape text reads "written as its `&lt;`-style unicode escape (`&lt;`, `&gt;`, `&amp;`)", which says each character is written as itself. That contradicts D1's own claim that `&lt;/script&gt;` cannot end the script, and its acceptance criterion that no literal `&lt;b&gt;` appears.
   - Evidence: `_lym/discussion/decision-record.md:43` (raw bytes confirmed with `cat -A`) against line 242.
   - Fix: write the escapes as `<`, `>` and `&`.

Tool calls:
- `grep os.Exit cmd/lym/main.go`: to see how lym sets its exit code, which my context did not show.
- `sed -n 40,64p cmd/lym/main.go`: to read main's exit-code path, for the same reason.
