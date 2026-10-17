# stencilstore dev-build warning prints on every creel verb

`stencilstore-dev-warning-noise` · Task · bug, stencil, creel · running · Finalize

Imported from [issue #702](https://github.com/Knatte18/loomyard/issues/702).

Seen in the creel re-tests of 2026-10-16 and 2026-10-17 (dev binary built by ./deploy-dev into .dev-bin): every lyx creel verb prints 'stencilstore: dev build does not refresh untouched stencils ...' on stderr. It is noise on every call, and it mixes into the stderr that creel verbs use for refusals and stall progress lines. Expected: the warning prints once per process that refreshes stencils, or only from verbs that read stencils, never from creel verbs.