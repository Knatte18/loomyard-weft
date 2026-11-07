# Registered hub templates in hubforge, built once per test binary

`hubforge-registered-templates` · Task · enhancement, fabric, testkit, git · running · Code-Burler-Fix

Filed from the Sketch `fabric-test-speed`.

Takes: hubforge's shape templates, relocation and copy in internal/hubforge/template.go
Delivers: a registration of key, base Shape and build function, built once per binary under a lock per key with a refusing testing.TB and the upstream-tip check, and a copy call that takes it; the PATTERN-test-isolation background gains the registered template
Exports: the registration type and its copy call in internal/hubforge

The part changes these packages:

- internal/hubforge (about 450 changed lines)
