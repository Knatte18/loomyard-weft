# Webster-Burler round 1: card 11's exact PATTERN text cannot fit the entry-line cap

Card 11 asks for one exact sentence after the semicolon in the `PATTERN-shuttle-stop` line, and to change nothing else.
With that sentence the line is 409 characters, but `internal/pattern`'s `MaxEntryLineChars` caps an entry line at 320, and `TestOverview_RealFilePassesCheck` enforces the cap.
The implementer had to reword the whole line, and the reviewer, checking against the card's text, raised the reword as F1 and proposed a fix that fails the plan's own verify.
Plan-Write could have caught this by checking a card's verbatim PATTERN text against the entry cap before approving the card.
