# Planning edits to platform-split Go symbols

A symbol with one body per GOOS (`internal/proc#StartTime`, `internal/gatecli#runChild`, `internal/gatecli#terminatingSignals`) is answered `ambiguous` by `lyx quarry resolve`, so a plan can neither `Edit` nor `Delete` it by glyph.
The only spelling the gate accepts is the file path, which means deleting whole files to retire such a symbol.

The gate then refused `create-already-exists` for a `plan:` handle whose name matched the symbol those deleted files declared, because the resolve pass reads the tree before any card runs.
The plan had to rename the replacements (`runTiedChild`, `terminatingSignalSet`) and edit their callers, a rename the design never asked for.
A gate that let a `Create` handle reuse a name whose every declaration the same card deletes, or a quarry answer that treats per-GOOS bodies as one declaration, would have avoided both detours.
