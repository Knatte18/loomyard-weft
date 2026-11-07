# internal/fabricengine anchor and origin fixtures onto registered templates

`fabric-anchor-origin-templates` · Task · enhancement, fabric, testkit, git

Decision record: [Sketch](decision-fabric-anchor-origin-templates.md)

Depends on: [`hubforge-registered-templates`](design-hubforge-registered-templates.md), [`fabric-test-folds`](design-fabric-test-folds.md)

Filed from the Sketch `fabric-test-speed`.

Takes: hubforge-registered-templates' registration and fabric-test-folds' folded tests
Delivers: newAnchorHub and newOriginHub, with originPair and endedHub where a fixed slug set repeats at least three times, built by registrations; originBoard runs on the copy
Exports: nothing

The part changes these packages:

- internal/fabricengine (about 150 changed lines)
