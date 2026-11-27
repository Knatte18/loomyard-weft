# Fabric core foundation: layout and core packages, snapshot, action list, executor loop and the layout retarget

`fabric-core-foundation` · Task · enhancement, fabric, testkit, pattern, cwd, loom, batten, landing, gate, orch, board, webster, drift, report · done

Decision record: [Sketch](decision-fabric-core-foundation.md)

Filed from the Sketch `fabric-git-free-core`.

Takes: today's internal/fabricengine and its importers; the decision record whole, its Added after Discussion entries included
Delivers: internal/fabriclayout with fabric's names, path constants, pure path functions, option and result types and the mutation entry as plain data; internal/fabriccore with the snapshot, phase and action list types and the outcome classes, and no rule yet; the phased snapshot reader with its dotgit and gitrepo reads and its git fallback for non-files ref storage, tested at tier1; the engine's executor loop; internal/testkit/fabrickit; every importer outside fabric that uses only layout names retargeted to internal/fabriclayout, about 470 files across the repository, with no alias left in internal/fabricengine; the leaf import-set tests and PATTERN-leaf-packages' sections for both new packages; the move of classify.go's path classification and the PATTERN-lym-files background edit that names its new home; the geometryTokenOwners and weftnameImportOwners entries for the new packages; every verb still on today's code. Under the parent's added entry 'The restructure cuts fabric's test code, it does not only move it', the foundation's own tests shrink: a test is kept only where it alone guards a rule, at the lowest tier that can check it, duplicates and the same rule tested at several tiers are deleted, not moved, and the change description lists every deleted test and, for each kept or moved test, the rule it alone guards
Exports: the fabriclayout names and types; fabriccore.Snapshot, its phases, the action kinds and outcome classes; the executor loop every later part drives; the fabrickit snapshot builder

The part changes these packages:

- internal/fabriclayout (about 1800 changed lines)
- internal/fabriccore (about 900 changed lines)
- internal/fabricengine (about 500 changed lines)
- internal/testkit/fabrickit (about 400 changed lines)
- internal/lymcwd (about 40 changed lines)
- internal/loomcli (about 250 changed lines)
- internal/battencli (about 200 changed lines)
- internal/fabriccli (about 100 changed lines)
- internal/landingshed (about 80 changed lines)
- internal/pairteardown (about 80 changed lines)
- internal/beamcli (about 80 changed lines)
- internal/hubgeom (about 80 changed lines)
- internal/gatecli (about 70 changed lines)
- internal/orchcli (about 60 changed lines)
- internal/webstercli (about 50 changed lines)
- internal/boardengine (about 40 changed lines)
- internal/driftengine (about 40 changed lines)
- internal/hubforge (about 30 changed lines)
- internal/testkit/envkit (about 20 changed lines)
- internal/testkit/boardkit (about 5 changed lines)
- pattern (about 60 changed lines)
