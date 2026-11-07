# internal/loomcli beamHub and newBadReedUpFixture onto registered templates

`loomcli-templates` · Task · enhancement, fabric, testkit, git · running · Drift-Baseline

Decision record: [Sketch](decision-loomcli-templates.md)

Filed from the Sketch `fabric-test-speed`.

Takes: hubforge-registered-templates' registration
Delivers: beamHub built by a registration, and newBadReedUpFixture's base, the hub with the corrupted reed line and the pair, as a template with each site's seed run on its copy
Exports: nothing

The part changes these packages:

- internal/loomcli (about 80 changed lines)
