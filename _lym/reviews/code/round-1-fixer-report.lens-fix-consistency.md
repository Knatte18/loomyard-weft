```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked the change set for twins left stale. I found no BLOCKING or MEDIUM findings and two LOW.

1. **LOW — `internal/reedengine/buttonclick.go`, `runButtonVia` doc comment (line 75).**
   - **Problem:** the doc lists the failures that "each show one message on the clicking client and run nothing further". The fix added a new one, `lym: <label>: the ask text holds a ','`, and the list does not name it.
   - **Evidence:** the new branch is `if asking && strings.Contains(ask, ",") { say(...); return }`. Line 75 still names only the missing-entry, spec, directory, anchor and failed-run messages.

2. **LOW — `internal/reedengine/doc.go`, RunButton "checks come in order" paragraph (line 87).**
   - **Problem:** it lists the click's refusals in order and omits the new one. The fix did add the comma refusal to the spec-contents paragraph at line 72.
   - **Evidence:** the click makes this check after the run-directory and anchor checks and before the prompt is raised. The paragraph ends with the run-directory and anchor checks.

Two things I checked and found consistent:
- **Config templates:** the `buttons:` comment in `template_posix.yaml:22` and `template_windows.yaml:22` says an ask holds "no comma". That still holds, because it speaks of the static text. It does not claim filled placeholder values are safe.
- **Decision-bearing options:** the `doc.go` line near 415 now names `@lym_button_answer`, which matches the line near 107.

Tool calls:
1. grep for the ask rules in docs and templates across the repo. My context lacked any non-Go docs or templates that restate the comma rule.
2. grep for "ask" and "comma" in `template_posix.yaml` and `template_windows.yaml`. The first grep matched them but showed no text, and my context lacked their contents.
