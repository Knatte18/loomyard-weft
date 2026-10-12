# Round 2 fixer-report — discussion

## Changes

- **F1 (BLOCKING), fixed differently.**
  The premise holds: `blankQuotedSpans` keeps `\` literal inside a single-quoted span, so `$'\''` closes at the escaped quote and the next `'` blanks the rest of the command.
  The proposed check, "a `$` followed by a single quote in the single-quote-blanked view", cannot fire, because that view blanks the quote character itself: `$'` reads as `$` before a space.
  D2 now reads the follower of a `$` the view keeps from the segment's raw text, which F2's deny-by-default rule covers, so `$'` is rejected.
  D2's line-58 paragraph says the command-wide substitution, redirection and backgrounding scans and the segment split rely on that rejection, and why.
  Goal 2 is reworded so it no longer claims two paths are the only ones.
  `cat $'\'' $(touch x) f` is in the false list, and `grep 'a$' f` and `grep "^a$" f` stay true.
  Scope Out now names teaching the quote scanner ANSI-C quoting as out of scope.
- **F2 (MEDIUM), fixed as proposed, with the raw-text follower from F1.**
  The dollar rule now rejects a kept `$` followed by anything other than whitespace, a double quote or the segment's end.
  The regex-anchor sentence is restated against it, `$~x` sits beside `cat $x` in the premise paragraph, and `cat $~x` and `cat $1` are in the false list.
  `grep a$ f` is added to the true list to pin the whitespace follower.
- **F3 (LOW), fixed as proposed.**
  The first `-no-cutoff` criterion now pins the untimestamped main-session transcript rule, and `cat "${x@P}"` and `cat $@` are in the false list.
- **F4 (NIT), fixed as proposed.**
  D2 now says "every output redirection but the two dropped stderr forms `2>&1` and `2>/dev/null`".
- The support log's Review rounds section records this round.

## Disputed

None.

## Deferred

None.
