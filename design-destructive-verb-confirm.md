# Destructive no-argument verbs act on a bare invocation

`destructive-verb-confirm` · Note · enhancement, fabric, board

## Seen

In the #512 incident (2026-10-10) a reviewer probing the CLI ran `lyx fabric unwire` and `lyx board retire-legacy` bare in a task pair; both acted at once, unwiring the pair and deleting legacy board files.
57243019b closed the route it took (global `--json` never runs a command), but any bare call of these verbs still acts.

## Idea

Give destructive verbs that take no argument (`fabric unwire`, `board retire-legacy`, `board prune`) an explicit confirmation flag, so a bare call prints what it would do and changes nothing.