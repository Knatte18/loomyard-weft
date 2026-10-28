# Comment lint ignores a legacy wrapped paragraph changed only by a word rename

`comment-lint-rename` · Task · enhancement, loom

## Comment lint flags every line break of a legacy wrapped paragraph after a pure word rename (was note `comment-lint-rename-breaks`)

Imported from [issue #881](https://github.com/Knatte18/loomyard/issues/881).

The comment lint judges a line break new when the word pair across it is absent from the base text.
A pure word rename inside a legacy fixed-column-wrapped comment paragraph changes the word pair at every break in that paragraph, so each break becomes a finding although the wrapping was never touched.

A mechanical binary rename run halted at its producer row (`anomaly: escalation-to-human`, `gate did not pass after 3 attempts`) on several hundred such findings in `gate-findings`.
The agent cleared them with a second scripted pass that rejoined each flagged paragraph and broke it at sentence ends.

This is abnormal because the findings point at comment wrapping the change never altered, and the halt cost the whole verify budget.

Suggested fix: compare word pairs modulo the rename, or ignore a break whose surrounding line text is otherwise unchanged apart from the renamed word.

Provenance: task `rename-binary-lym`; notes `Darn-Write.md`, `loom-halt.md`.

