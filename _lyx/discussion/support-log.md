# Support log — stencilstore dev-build warning on creel verbs

## Interview

A darn run holds no interview; the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- Downgrade the dev-build refresh notice from Warn to Info: rejected, because the notice tells a dev-build tester that producers read older stencil copies, which matters for every stencil-reading verb, and it would still print under `-v` from creel.
- Make `skipStencilSeed` walk the command's ancestors so the group annotation covers its children: rejected, because it changes the per-command contract of the annotation for every group in the binary.
- Print the notice only from verbs that read stencils by moving the seed pass out of the root pre-run: rejected, because it touches every stencil-reading module for a bug confined to creel.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Does the warning print on every creel verb or once per seed write? Self-pick: once per seed pass that writes, per the sandbox trace logs; the fix covers either reading. Resolved.
- Q: Does the hidden `daemon` command opt out too? Self-pick: yes, it is a long-lived process that reads no stencils. Resolved.
