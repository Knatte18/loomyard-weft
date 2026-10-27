# Markdown link guard skips the verbatim data of research reports

`docs-link-skip-bench-data` · Task · bug, testkit · running · Darn-Write

## Seen (2026-10-27)

`TestEnforcement_MarkdownLinks` (`internal/lymcwd/docslink_test.go`) fails on `main`: it reports broken `pattern/PATTERN-*.md` links in files under `docs/research/*-2026-10-27/data/`, such as `data/prompts/step-1-explore-fan.md` and `data/reviews/P4/handler.md`.
The bench reports keep prompts and reviewer output verbatim in their `data/` directories, and those texts link to `pattern/` relative to the repository root, so the links do not resolve from where the file sits.
The bench landings changed only `docs/`, so no gate on their way ran the guard; every run's verify now meets it.

## Fix (operator, 2026-10-27)

- The guard skips every file under a `data/` directory of a `docs/research/<report>/` directory: that is captured verbatim data, not documentation, and its links are never rewritten.
- The allowlist entry for `docs/research/codex-astra-solo-bench-2026-10-25/data/reviews/P4/opus-medium-r1.md` goes, since the skip covers it and the guard would report it stale.
- A subtest pins the skip: a broken link under `docs/research/x/data/` is not reported, and one in `docs/research/x/report.md` still is.
- The guard's doc comment and the Markdown Link Integrity text in `PATTERN.md` or its `pattern/` background file, wherever the guard's scope is stated, say that `docs/research/*/data/` is outside it.
