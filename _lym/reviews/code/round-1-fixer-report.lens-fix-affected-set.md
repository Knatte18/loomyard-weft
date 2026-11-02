**Unchecked:** F1, F5 and F6. Their premises are single sites, so no set search applies.

**F2, callers of `readDeployState`:** all callers take the new `readInstalled` parameter.
- `deps.go:162` passes `runningLymBuild`.
- `export_test.go:19` passes it through from the test.
- `deploy.go:76` is the definition, and `doc.go:63` only names it.

No finding.

**F3, packages `adoptcli` imports:**
- The fix added `battenshed` and `buildvcs`, the two imports the diff introduced, so the finding as raised is covered.
- The line was already a partial list before this change. Non-test imports such as `orchcli`, `shedrun`, `shedengine`, `gateslot`, `hubgeom`, `hubreconcile`, `dobbyengine`, `reedengine` and `arbiterstop` are still not on it, so it reads as "the takeover functions" rather than a full list.
- That gap predates the fix and the fix did not worsen it, so under the rubric it is not a finding.

**F4, sites that describe the adopt steps:**
- `docs/overview.md:448-450` now has the deploy step.
- `internal/adoptcli/doc.go:59-68` and the help text at `cli.go:41` and `cli.go:47` already describe it.
- `pattern/PATTERN-owner-machine.md:7` mentions `lym adopt` only as the owner-taking path and does not walk the steps, so nothing there went stale.

No finding.

**Result:** no findings.

Calls made:
1. Listed the non-test imports of `adoptcli`, every reference to `readDeployState` and `ReadDeployState`, and the deploy lines in `doc.go` and `cli.go`, to check F3 and F2. My earlier broader grep returned too much, so I narrowed it.
2. Grepped the `docs` and `pattern` trees for `lym adopt`, to find other sites that describe the adopt steps for F4.
3. Grepped `PATTERN-owner-machine.md` for adopt, to see whether that hit walks the steps.
