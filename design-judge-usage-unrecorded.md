# Judge and seed passes are missing from the per-round usage files

`judge-usage-unrecorded` · Note · enhancement, burler, loom

## Seen

The haiku-roles analysis (`docs/research/haiku-judge.md`, 5459376bb) had to measure judge and seed cost from session transcripts, since the per-round usage files have no judge entry.

## Idea

Record judge and seed passes in the usage files like other roles, so the token report per role shows them and a Haiku trial can be compared without reading transcripts.
