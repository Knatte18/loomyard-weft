```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Task: check the fix for F1, the only finding, by rerunning its search for stale "drop on any cause" and "local-path origin" claims.

- Searched all `.md`, `.go` and `.yaml` files, excluding `_lym/reviews`, for "unreadable machine record", "local-path origin", "Whatever the cause", "dropped, not kept", "drops that entry" and "drops an entry".
- No stale claim is left. The two hits outside `summary.md` are unrelated (`contracts/specs/loom-discussion-spec.md:125` and `internal/lenses/template.yaml:8`).
- The "What changes" bullet in `summary.md` says an entry "that resolves to none" is dropped. That is consistent with the rewritten design-decision bullet, so it needs no change.

Tool calls:
1. Ran `grep -rnE` over the repository for the stale phrases, to find any other site repeating F1's two wrong claims. My context held only `summary.md`.
