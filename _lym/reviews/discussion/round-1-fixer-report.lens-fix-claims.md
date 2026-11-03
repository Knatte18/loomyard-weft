```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: I checked the claims the fix added or rewrote against the code at HEAD.

- **MEDIUM.** The Land not-live way forward in D2 says: "With no archive tag on origin the branch was deleted outside fabric, which archives before every removal."
  Fabric does not archive to origin before every removal.
  `RemoveLocal` (`internal/fabricengine/remove.go:165-172`) puts only a local `set-aside/<slug>/<sha>` tag on the weft tip and pushes nothing. `internal/pairteardown/teardown.go:209-210` routes any `Request` with `Local` set to it.
  A pair carrying the local-only mark "commits and archives nothing" (`remove.go:154`).
  `archiveWeftTip` also skips when the weft repo has no origin (`archive.go:78-82`).
  So a missing archive tag on origin can come from a fabric removal, and in the `RemoveLocal` case the records sit under a local `set-aside/` tag that the stop's restore never names.

- **MEDIUM.** D6 says: "In the origin-only path, the lease is the origin tip the path read, which its archive already covers."
  The archive and the origin-only delete read origin separately.
  `archiveWeftTip` reads the tip through `resolveArchiveTip`'s own `ls-remote` early in `removePair` (`remove.go:269`, `archive.go:149`).
  `removeSibling`'s origin-only branch reads it again later with `remoteHeadTip` (`remove.go:495`).
  The teardown lock does not stop another machine from moving origin between those two reads, so the tip the path read is not guaranteed to be archived.
  Leasing on the tip the archive step read would make the claim hold.

- **LOW.** D2's newest-archive-tag pick says the newest tag is "the archive of this run's removed pair."
  Under D4 step 3 and D6's sweep rule, one removal can push two tags: one at the local tip and one at a diverged origin tip.
  "Newest commit" can then pick origin's diverged tip, not the local tip that holds this machine's run records.
  The claim that this restores "the child's run records" doesn't hold in that case.

- **LOW.** D7's Bound says "a pass's `Add` that meets it is transient under D2, so dobby retries it."
  D2 only classifies the answers of D1's adopt-only re-adoption.
  The Worktree-Create row's `Add` is also "a pass's `Add`", and no decision gives it D2's transient handling.

- **LOW.** The Constraints evidence says "this removal had finished 19 s before `Add` began."
  The 08:18:20 timestamp is the cancel's run-directory commit (`15e61b6a75`), not the end of the removal.
  The 08:18:39 timestamp is `Add`'s origin-record commit, which comes at the end of `Add`.
  Run B's owner claim at 08:18:36 shows `Add` started later than 08:18:20 but earlier than 08:18:39, so "19 s" isn't backed by the evidence; "finished before `Add` began" is.

Other claims checked hold:
- `Add`'s anchor and source-dirt refusals.
- `removeSibling`'s tolerance of a failed local weft-branch delete, and local-branch liveness without reading tags.
- `checkRunStatus` resuming a failed status.
- `childReachedParent`'s true cases.
- `liftSlugPauses`, `resolveArchiveTip` fetching by SHA, and the 62 s of archive-tag delays.
- The step-11 leased delete.
- The 08:14:52, 08:14:59, 08:18:39, 08:18:44 and 08:18:46 log and commit times.
- The `--skip-connectivity-check` doc text.
- The rule-text cap of the PATTERN change.

Tool calls:
1. grep for `RemoveLocal`, `DropLocal` and the cancel's step order: my context lacked who calls `RemoveLocal` and that `lym adopt` reaches `DropLocal`.
2. `internal/pairteardown/teardown.go:195-215` and a grep of `DiscardTaskBranch`: my context lacked when a teardown takes the local-removal path.
