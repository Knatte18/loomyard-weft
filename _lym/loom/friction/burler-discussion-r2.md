# Fixing discussion round 2: backslash-u escapes do not survive the edit path

F7, like round 1's F14, asked the record to hold the literal text of Go's JSON unicode escapes for `<`, `>` and `&`.
Text typed as a backslash, `u` and four hex digits reaches the Edit tool already decoded into the character itself, so the edit is a no-op or writes the bare character.
Round 1's fixer reported the fix as made, and it never reached the file.

This round it surfaced only because Edit refused an old_string equal to its new_string, and `od -c` confirmed the bytes.
The record now spells the escape out in words.
A review asking for such literals should expect this, and a fixer should check the raw bytes after the edit.
