# internal/battencli beamHub onto a registered template

`battencli-templates` · Task · enhancement, fabric, testkit, git · running · Open-Window

Decision record: [Sketch](decision-battencli-templates.md)

Depends on: [`hubforge-registered-templates`](design-hubforge-registered-templates.md)

Filed from the Sketch `fabric-test-speed`.

Takes: hubforge-registered-templates' registration
Delivers: battencli's own beamHub, a beam, a member and a plain pair, built by a registration; notices_integration_test.go untouched
Exports: nothing

The part changes these packages:

- internal/battencli (about 50 changed lines)
