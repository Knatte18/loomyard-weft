# Webster plan validation refuses a card path with a trailing '#' anchor that Plan-Bouncer accepted

`card-path-empty-anchor` · Task · bug, webster, parsers · running · Record-Write

Imported from [issue #34](https://github.com/Knatte18/loomyard/issues/34).

What happened: `lym webster run` failed the run (producer-hard-failure) with two blocking findings on one card: `path-missing` and `glyph-not-found`. The card path was a markdown file followed by a bare `#` (an empty heading anchor), e.g. `<dir>/README.md#`. The same refusal recurred in a later run, once with a repo-relative path and once with a path relative to the module directory.

Why abnormal: the plan passed Plan-Bouncer and Plan-Burler review, then died at Webster after the plan was already accepted. Either the plan check should reject the malformed path early, or the Webster path parser should split the `#anchor` suffix from the file path and treat an empty anchor as the whole file.

What to do: share one path-with-anchor parser between the plan bouncer and `lym webster` validation, and make the bouncer reject what Webster rejects.

Provenance: task `commonlib-fluid-report-selection`; notes loom-halt-3.md, loom-halt-5.md.
