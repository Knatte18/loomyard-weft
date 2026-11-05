# Beam landing overwrites a reopened PR's hand-written body and title with a flat title list

`beam-landing-overwrites-pr-body` · Note · bug, pawl, landing

Imported from [issue #56](https://github.com/Knatte18/loomyard/issues/56).

Seen in hub Models-LYMHUB, beam duplication-and-import-followups, 2026-11-04.

The beam's pawl run blocked because its recorded PR #267 had been closed; the orch reopened it, set a full body listing every merged sub-PR with link, merge time and summary, and a new title, then ran lym loom resume. 33 seconds later the run rewrote the PR (GitHub edit history: 18:19:16 orch body, 18:19:49 lym body): the title went back to the original and the body became the summary.md flat list of sub-PR titles, with no links or explanations. It also rewrote _lym/landing/summary.md to that flat list, dropping the dated list the operator had asked for earlier.

Expected: the landing does not regenerate a body over an existing PR body it did not write, or it builds the sub-PR list with links, merge times and summaries as operators want for a beam PR.

Workaround: the orch restored the body and title from the PR's edit history with gh pr edit, and the pr-reviewer put the dated list back into summary.md.