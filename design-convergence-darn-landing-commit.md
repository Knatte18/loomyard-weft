# report convergence: 'claim one landing commit' error for darn archive tags

`convergence-darn-landing-commit` · Task · bug, report, batten · running · Merge-In

Imported from [issue #831](https://github.com/Knatte18/loomyard/issues/831).

Seen (loomyard hub, Neve, 2026-10-22): `lyx report convergence --since 2026-10-20` printed a "claim one landing commit" error in `findings` for 9 darn runs' archive tags (research-in-discussion, shuttle-open-agent-defaults, decision-add-notifies, gate-tier-verbs, kickstart-pack-remove, board-entry-handles, gate-test-runtime-caller-paths, report-timing-unreadable-journal, darn-hard-cap-default-2), all mapped to the same commit de6b8465f, although each landed its own commit on main.

Workaround: none; those runs' landing data is missing from the report.

Expected: each darn run's archive tag maps to its own landing commit, as loom runs' tags do.