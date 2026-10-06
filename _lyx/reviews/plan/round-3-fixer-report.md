# Round 3 fixer report

## Fixed

- **F1** (stale card number): `prune-procedure` step 1 now names `rerun-measure-after` instead of a card number.
- **F2** (skill-loading reason outside the closed set): step 6 now lists the skill-loading reason as a fifth class, bounded to the tests `ownership-boundary` lists; card 21's keep check names the same set.
- **F3** (loop exit): step 8 requires a keep directive whose reason is one of the step 6 classes, and step 6 states that no other text satisfies the loop.
- **F4** (card 16 too wide): split into `16-fabric-geometry-prune.md` (fabric CLI, `hubforge`, `hubgeom`, `standalonegeom`, `standalonestate`, `lyxcwd`, `pairteardown`, `weftname`) and `17-git-fs-prune.md` (`gitexec`, `gitkit`, `gitrepo`, `gitoracle`, `githubclient`, `fslink`, `fsx`, `lock`, `proc`).
  The later cards moved to 18–21: files, headings, `Commit:` subjects, the Card Index, both `Applies to:` ranges and card 1's two references to the after-measurement card were updated.
  The set of targeted packages is unchanged and no package is targeted twice, checked by comparing every card's `Edit` list with the earlier list.
- **F5** (card 20 labels and leaf rule): card 20 names `batcher` and `shell` as registry and seam packages and carries their pattern checks; the leaf closed-set rule now sits on cards 16, 17 and 20, where the leaf packages `standalonestate`, `gitkit` and `friction` live.

## Deferred

None.
