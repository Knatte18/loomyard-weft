```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set for sibling or twin inconsistencies (fix-consistency lens).

Findings: no findings.

- Every `Strands()` call under `s.do` is gone. The only one left in the creelengine store is `readStrands`, which sits behind `listStrands`, outside the lock. The other callers are `deliver.go`, `server.go`, `sweep.go:114` and the chaser's `probe`, none of which hold the store mutex as far as this grep shows.
- `settleUnansweredLocked` and `noticeUnansweredLocked` have no callers besides `stepChases` and `releaseGoneHolder`. Both pass the listing, so no twin keeps the old signature.
- The error text is unchanged. `readStrands` wraps with the same "creel: could not read the hub's strands" message the inline code used.
- No other live text refers to the removed shuttle send. The only repo hits are `docs/research/codex-feasibility.md`, a research note, which falls outside the fix's reach. The generic "shuttle sends" hits in shuttleengine are about other things.
- `NotifyAsker` is called with the same arguments in all three wirings, so the new test's string match holds in each.
- The new test is in `package orchcli_test`, and no other orchcli test reads sibling source through `../`. That is a new pattern, not a conflict with an existing one.

Tool calls:
1. Ran grep for "shuttle send" in `.go` and `.md` files and `../` paths in orchcli tests, and for `Strands()` in creelengine, to find twins of the changed text. My context held only the three edited files, not the rest of the repo.
