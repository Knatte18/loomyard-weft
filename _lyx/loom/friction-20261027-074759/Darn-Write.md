# Comment lint flags legacy wrapped comments a mechanical rename touches

The comment lint judges a line break new when the word pair across it is absent from the base text, so a pure word rename (`lyxcwd` to `lymcwd`) inside a legacy fixed-column-wrapped paragraph makes every such break a finding.
The rename run halted after three gate attempts on 286 findings in comments whose wrapping it never changed.

A second scripted pass that rejoins each flagged paragraph and breaks it at sentence ends cleared them, but a lint that compared word pairs modulo the rename, or ignored breaks whose surrounding line text is otherwise unchanged, would have spared the halt.
