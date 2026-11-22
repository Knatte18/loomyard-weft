# A stale records tracking ref no longer blocks a cancelled slug's rerun

A slug cancelled with `lym batten cancel --remove` and released again could not start:
`lym batten run` refused it with `the code tip <sha> the run's last transition names is not on origin yet`,
though origin held the tip.
The records repository still held `origin/<slug>-weft` from the cancelled run after origin had deleted that branch.
`fabricengine.OriginCodeTip` reads the fetched records ref with no network call,
so it found the old run's last transition there and judged its code tip missing.
Nothing on the start path refreshed the ref, and no fetch dropped it, since `gitrepo.Fetch` did not prune.

## What changes

- **`gitrepo.Fetch` prunes.**
  It runs `git fetch --prune`, so every plain fetch drops the tracking ref of a branch origin deleted.
  That covers the records fetches of the pass catch-up, `FetchPairTips` and `lym fabric pull`,
  and the code-side fetches the same way.
- **batten's act check fetches before a code-tip wait stands.**
  `hubRunVerdict` in `internal/battencli/arm.go` now goes through `judgeAfterCodeTipFetch`:
  a verdict whose reason is `code_tip` fetches the run's branches, the pair's when it stands here and the prime's records otherwise,
  and judges a fresh read.
  A stale records ref is pruned by that fetch, so the second judgement acts.
  A tip still missing waits as before, and a failed fetch keeps the first verdict.
- **The code-tip refusal names the real cause when no pair stands here.**
  `runfsm.Inputs` and `runfsm.Verdict` carry `CodeTipPairAbsent`, which `pairteardown`'s run reader sets for a run with no pair on this machine.
  The refusal then says the transition may come from a records branch origin deleted, as `lym batten cancel <slug> --remove` does,
  and names `lym fabric pull` in the prime first, with the push on the owing machine only if the refusal stays.
  A run whose pair stands here keeps the old wording.

## Design decisions

- The board entry offers two fixes for the stale ref: drop it in the cancel and remote cleanup, or make the records fetch prune.
  This change takes the prune.
  A branch can vanish from origin by routes no local teardown sees, such as another machine's cleanup, and a prune covers them all;
  the teardown paths already drop the ref when their own delete succeeds.
- `OriginCodeTip` keeps reading only fetched refs.
  Deciding whether origin still holds the records branch needs a network call, which belongs to the caller about to act,
  so the fetch sits in batten's act check, not in the reader.
- The dobby's own code-tip settling is unchanged: it covers runs with a batten record and a pair here,
  and a removed slug's fresh start goes through `lym batten run`, which this change makes fetch.

## Tests

- `TestFetch_BranchDeletedOnRemote_DropsItsTrackingRef` (gitrepo, tier 1): a clone that fetched a branch loses its tracking ref once another clone deletes the branch on the remote, and keeps the live branch's ref.
- `TestJudgeAfterCodeTipFetch` (battencli, tier 0): a code-tip wait fetches and acts once the fresh read is clean, waits when the tip stays missing, keeps the wait when the fetch fails, and any other verdict fetches nothing.
- `TestAct` (runfsm) gains a row pinning the refusal text for a missing code tip with no pair here.

## Merge danger

**Door:** two-way
**Blast radius:** gitrepo
