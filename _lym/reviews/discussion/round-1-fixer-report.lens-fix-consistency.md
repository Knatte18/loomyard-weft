```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the change set agrees with the decision record and support log around it.

**[LOW]** `_lym/discussion/support-log.md:80` (Q5) still says profiles are carried "by a reinterpretation".
- **Changed text:** D6 is rewritten to "configsync moves profiles.yaml's blocks before it follows the template". The Auto-mode assumption at the old line 170 was changed to match.
- **Twin left as it was:** the support log's question ledger still reads "Q5 (auto-pick): how are file layers carried? By a `configsync` move table on every reconcile, with `profiles.yaml` by a reinterpretation and board `run_settings` by a once-per-hub walk step."
- **Evidence:** this pick now contradicts the record's D6. The "once-per-hub walk step" part was already stale before this round.

Outside my lens: D4's new closing sentence says that in all three file layers an unknown key "otherwise only warns", and that the warning names both reconcile commands. `warnUnknownKeys` (`internal/configengine/config.go:306-314`) warns only for the default and worktree layers; it skips the local layer. The fix-claims lens should weigh this.

Everything else agrees with the change set:
- **D7:** its shared-helper sentence still fits D6.
- **Constraints:** the walk-order line is updated.
- **D5 bound and D7 bound:** both match D4's per-site ways forward.
- **Criteria:** the D4, D5, D6 and D7 criteria match their decisions.
- **Plan-writer note:** it matches D4.
- **PATTERN changes:** the line is still consistent.

Tool calls:
- grep of `_lym/discussion/support-log.md` for reinterpretation, D6, D7 and reconcile mentions: my context held only the log's first 30 lines.
