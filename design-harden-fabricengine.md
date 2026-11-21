# Hardener campaign on fabric, one-repository layout

`harden-fabricengine` · Task · bug, fabric

Depends on: [`hardener-self-charter`](design-hardener-self-charter.md)

Harden only the one-repository hub layout of fabric: records and board under origin's `refs/lym/` (from 57015a52), the clone that seeds or joins that layout, its refspecs, push and pull, teardown and archive tags in that layout. The rest of fabric, the two-repository layout included, is out of scope.
