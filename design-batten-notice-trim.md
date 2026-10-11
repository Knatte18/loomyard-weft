# Batten notices drop the attach clause and use hub-relative paths

`batten-notice-trim` · Task · enhancement, batten, orch

## Seen (operator, 2026-10-11)

A batten notice to the orch ends with `; cd <absolute pair path> && lyx reed attach` (`internal/battenshed/notice.go:150`, `renderNotice`), and its report clause carries the stop report's absolute path.
The orch knows how to attach to a pair, so the attach clause is noise, and absolute paths repeat the hub directory every orch already works in.

## Wanted

- Drop the attach clause from every batten notice.
- Render every path in a notice relative to the hub directory (the directory holding the prime and the task pairs), for example `fixer-critical-of-findings/_lyx/shed/fixer-critical-of-findings/drive-reports/drive-report-….md`.
- Update the notice tests and the docs that quote a notice line.
