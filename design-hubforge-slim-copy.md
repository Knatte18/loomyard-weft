# A slimmer hubforge.CopyHub: packed templates, no walk per copy, two-process binding fold

`hubforge-slim-copy` · Task · enhancement, fabric, testkit, git · running · Describe

Decision record: [Sketch](decision-hubforge-slim-copy.md)

Filed from the Sketch `fabric-test-speed`.

Takes: hubforge-registered-templates' build path in internal/hubforge/template.go
Delivers: CopyHub without the fingerprint walk, SharedHub keeping it, repack and pack-refs at build time for shapes and registrations, and commitRelocatedBinding without its git add
Exports: nothing new

The part changes these packages:

- internal/hubforge (about 120 changed lines)
