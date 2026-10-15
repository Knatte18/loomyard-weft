# Round 2 fixer report — discussion

## Changes

- F1 (MEDIUM), fixed as proposed: D1's last Decision sentence no longer gives the Plan host a value for the relaunch marker.
  The premise holds: `planLensDirective` sets `WriterFork`, and `twilldirective.Directive` fills `twill-relaunch.md` only when `WriterFork` is nil.
  The replacement wording is the `fix` key's, set as its own sentence after "`ReviewedFiles` is unchanged, so the forks still review two files." rather than joined to it by a comma.
  Scope In is unchanged.
- F2 (LOW), fixed as proposed: D3's Decision now says "a `..` segment between `/` or `\` separators", and its Bound now says the path rule keeps an entry's backticked path inside the worktree and the reason text is prose and unchecked.

`support-log.md` holds no wording either finding touches, so it is unchanged.

## Disputed

None.

## Deferred

None.
