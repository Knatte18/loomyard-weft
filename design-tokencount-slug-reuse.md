# Tokencount mixes older runs' transcripts into a reused slug

`tokencount-slug-reuse` · Note · enhancement, shed

Seen at run-time-by-role PR-Gate (PR #550): a slug reused across runs pulls older sessions' transcripts into its time, as it already does for tokens.

Idea: bound a run's transcripts by its span (seed start to last history end) so a reused slug counts only its own run.