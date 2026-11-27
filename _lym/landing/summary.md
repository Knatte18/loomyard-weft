# shellguard: deny in-place edits that follow a closed heredoc

The `shell-edit` guard now refuses `sed -i`, `perl -pi` and `tee` into a source file
on the lines after a heredoc's closing line.
Before, it read nothing past the heredoc operator's line,
so writing a script with a heredoc and running the editor on the next line passed.

Heredoc bodies are still skipped, with the false denials listed under the design decisions.
The script of a `sh`, `bash`, `dash`, `ksh` or `zsh` `-c` after the closing line is read too, one level deep.
The new reach covers only the in-place edit match; the redirect match and the other command-shaped rows are unchanged.

Closes #268

## Design decisions

- `grep -E` has no backreference, so the closing line is tied to the operator's word
  only for the words in `heredocDelimiterWords`, each rendered as its own alternative.
- For any other delimiter word, a body line holding a bare word alone is taken for the closing line.
  A later line that opens with an in-place edit spelling is then denied, even in prose that only names the command.
- The same false denial follows a tab-indented line holding the listed word of a `<<` operator,
  and a `<<` that shifts by a name.
- A delimiter word may not open with a digit or a dash, so an arithmetic shift such as `1<<2` is no operator.
- These shapes still let an in-place edit through:
  - a heredoc with no closing line;
  - two heredoc operators on one line, since each body cannot be tied to its operator;
  - a delimiter word that opens with a digit or a dash;
  - a quoted delimiter word holding a space or a character that ends an unquoted word;
  - a closing word holding anything but letters, digits, `_`, `.` and `-`;
  - a later line of a `shell -c` script, after a heredoc closed inside that script.
- `notWordFrom` now takes the word-breaking characters and the first-character exclusions as arguments,
  so it serves both the `gh` words and the delimiter words.
- The Claude settings golden is regenerated from the new pattern.

## To check by hand

- A heredoc body line holding a bare word alone can end the skipped body
  (`closedHeredocPattern` in `internal/shellguard/rows.go`).
  The listed delimiter words are tied to their closing line; confirm the remaining false denial for other words
  is acceptable.
- A `shell -c` script around or after a heredoc was not read
  (`inPlaceEditAfterClosedHeredocPattern` in `internal/shellguard/rows.go`).
  A script after the closing line is now read;
  confirm that leaving a heredoc closed inside the script unread is acceptable.
- The final rendered hook pattern was not timed under `grep` on a large command with several heredocs.

## Merge danger

**Door:** two-way
**Blast radius:** sessions

A wrong pattern changes which Bash commands the guard refuses in lym sessions.
