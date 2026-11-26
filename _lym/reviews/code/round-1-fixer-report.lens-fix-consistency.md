```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the change set (agreement with siblings and cross-references).

1. **LOW** — `_lym/landing/summary.md`, the two sentences the fix added or lengthened.
   - They pass the 120-character line limit the rest of the file follows.
   - They are the "Its not-attached and unreadable-row errors now read …, and a failed listing's error gains `list clients:`." line and the "Both rows are guardrails, not barriers: … as each row's comment states." line.
   - Evidence: the prose rule says "Keep each line under 120 characters of text", and neither line holds a code span that passes the limit alone. Break the first before ", and a failed listing's" and the second after the colon.

2. **LOW** — `internal/shellguard/rows.go`, the `session-bus` row's `Forms`, unchanged beside the changed pattern.
   - `{"dbus-send"}` still declares every `dbus-send` as a denied form, but `dbus-send --system` now passes.
   - Evidence: `Row.Forms` is documented as "the literal command forms the row denies, each a whole token sequence". The sibling `busctl` entry is narrowed to `{"busctl", "--user"}` for this reason.
   - A positive "without `--system`" form cannot be expressed as a token sequence. Keeping the entry and saying so in the row comment, or adding `{"dbus-send", "--session"}` in its place, are the options. The first keeps the no-denied-recovery scan's coverage of a bare `dbus-send`.

3. **LOW** — `rows.go`, the `session-bus` row comment ("So do systemctl --user, and busctl and dbus-send on the system bus") and the `sessionBusCommandPattern` comment.
   - Neither says that a command carrying `--system` as well as `--session` or `--bus=…` now passes, although it need not reach the system bus. From memory, `dbus-send` takes the last bus flag; I did not verify that.
   - Evidence: the pattern matches only when no argument word is `--system`, so `dbus-send --system --session …` is let through.
   - `doc.go` says each pattern's comments state what it lets through.

Outside my lens, for the correctness lens or the fixer: a tab-separated `dbus-send<TAB>--session` appears to pass now. The payload encodes the tab as `\t`, which neither the new `space` nor `commandEnd` matches, whereas the old `wordEndPattern` matched the backslash. I derived this from the pattern and did not run it.

The other cross-references agree: the steer, `doc.go`, the golden, the machine-sleep comment, the summary's `dbus-send` bullets, the `hub-tmux-server` comments and the sampler stencil.

Tool calls: none.
