```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F6) and report the cases where a proposal is wrong or breaks something.

**Findings**

1. **MEDIUM** — `pattern/PATTERN-pane-text-injection.md` line 3, the reworded opening: "closes a menu only past the quiet-window guard: no operator was active in the pane within `operator_quiet_s`."
   - The rule now says every menu close goes through the quiet-window guard.
   - The guarded rewind-picker close does not do that. `closeRewindPicker` in `internal/shuttleengine/interruptguard.go` captures the pane, plays one key, sleeps the settle and re-captures. It never calls `operatorActivityHold`.
   - `closePickerBeforeSend` calls it directly. The PATTERN bullet that describes the rewind close, "The guarded picker close is one of two exceptions", lists no quiet-window check either.
   - So the reviewer's prescribed rewording ("say the quiet-window guard is what protects an operator's open menu") is false for one of the two exceptions it now covers. A reader of the rule would expect a guard the rewind close does not have.
   - The sentence should state the quiet-window guard for the dismissal only, or admit that the rewind close has none.

2. **LOW** — `internal/shuttleengine/gate_noinputbox_test.go`, row "a delivered re-prompt and the next arrival restart the count".
   - This row does not pin the reset on a delivered re-prompt (the one at `wait.go:582`) that finding F3 names. Its own comment says the delivery and the arrival each restart the count.
   - The arrival reset at `wait.go:347` hides the delivery reset, because after a delivery the boundary closes until a new arrival.
   - The fixer's mutation run supports this. With the arrival reset removed, this row still passed, because the delivery reset covered for it.
   - Removing only the delivery reset also leaves the row green, because the arrival reset covers for that.
   - The row therefore asserts the combination, not either reset. The test's doc comment still claims each case.

3. **LOW** — `pattern/PATTERN-pane-text-injection.md`, the new dismissal bullet's last sentence: "a gated run's re-prompt loop ends after a bound of holds by a pane that still shows no input box."
   - This is gate-loop behaviour inside a rule about pane key injection.
   - It cannot change what the rule allows, and it duplicates `wait.go` and `doc.go`.
   - Per the prose skill's rule against pinning perishable specifics, it goes stale silently if the bound's mechanism changes.

**Proposals with no finding**
- F4: marking a failed dismissal key `noInputBox` is sound. The active-operator hold stays unbounded, as the review required.
- F5: asserting `NoInputBox` on the conditioned-send busy error breaks nothing.
- F6: the `─` fixture is read as a picker only because no bottom rule sits below the options. `TestDismissiblePicker` now pins that, and I see no breakage.
- F2: the guard and settle test breaks nothing.

**Tool calls**
1. `grep` plus `sed` on `closePickerBeforeSend`. I read it to check whether the rewind close calls the operator-activity guard, because my context held only `closePickerBeforeSend`'s name, not its body.
