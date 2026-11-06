```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

F1 is the only finding. Its set is the window state that `noticeDropLocked` changes before the post. A grep of `internal/` shows `noticeDropLocked` is the only writer of `window.notices` and `window.undisclosed`. `dropLocked` is its only caller, and `postDropNoticeLocked` is called only from `noticeDropLocked`. The restore at `receive.go:348-349` covers every field the function mutates before the post: `notices` and `undisclosed`. The window's `start` is untouched by the post path.

Findings I left unchecked: none.

Tool calls (1):
1. `grep` over `internal/` for `postDropNoticeLocked`, `noticeDropLocked`, `.undisclosed` and `window.notices`, to confirm no other site spends the window before a post. My context did not hold callers outside `receive.go`.
