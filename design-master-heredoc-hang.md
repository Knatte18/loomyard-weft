# Webster Master hangs background shells with a flattened heredoc

`master-heredoc-hang` · Note · bug, webster, shuttle

Seen 2026-10-05 in two Webster Masters at once (`orch-cycle-hardening` and `webster-audit-recovery`), each with a background shell hung for about 80 minutes:

```
cd internal/shuttleengine/claudeengine && cat > /tmp/x 2>/dev/null; python3 - <<'EOF' 2>&1 | head -1 EOF echo ok
```

The heredoc arrived flattened onto one line, so `EOF` never closes it and `cat > /tmp/x` reads stdin forever.
It is the same shape as #361 in `board-model` (`python3 - 2>/dev/null; cat > /dev/null <<'EOF' … EOF`), so it is a recurring Master habit, not a one-off.
In `orch-cycle-hardening` it held the verify gate from 12:51 until the hub killed the `cat` at 12:56; the gate then ran verify.

`webster-audit-recovery` bounds the wait and gives every agent Bash call `/dev/null` as stdin, which ends the hang but not the habit.
Open: why Master writes a multi-line heredoc that reaches the shell as one line, and why it reaches for `python3` and `/tmp` at all; the fix belongs in the Master stencil or the tool guidance it gets.
