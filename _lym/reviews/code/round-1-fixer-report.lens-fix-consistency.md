```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1–F4 change set.

no findings

I checked these and they agree with the change:
- **`strandsElsewhereLocked`:** both callers (`lifecycle.go` Status and `reconcile.go` `reconcileLocked`) take the new two-value return, and the reconcile comment matches what it does.
- **Cause list:** the `strand_down` help names the new `DownPanePlacementUnread` alongside every other cause an `orch status` envelope can carry.
- **Related prose:** the `Elsewhere` field doc, the `strand_elsewhere` help, the `orchroute` and `reedengine` package docs and the `OrchLive` comment still hold. No `.md` file lists the causes.
- **Log line:** both `Live` branches use `downReadingLogMessage`, and no other copy of the literal turned up in the grep.
- **Test:** the header comment and rows agree with the new failed-lookup behaviour and the unbound strand.

Tool calls (1 of 6):
- One Bash call, `git diff` of the `doc.go` files and `orchcli/live.go` plus a grep for sibling wording. My context lacked those hunks and any repo-wide list of prose describing the changed behaviour.
