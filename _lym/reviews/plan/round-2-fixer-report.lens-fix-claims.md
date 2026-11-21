```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added holds against the code at HEAD (fix-claims lens).

**MEDIUM: the "live local branch" bullet can't be carried out as written**

- **Changed text:** `_lym/plan/07-pair-lifecycle-namespace.md`, `TestOneRepositorySecondMachine_AdoptPullAndReplay`, the new bullet "live local branch: with the second machine's records branch present locally and origin's refs/lym/heads/&lt;branch&gt;-weft ahead of it, `Topology.Add` of the slug reaches `probeLiveLocalWeft`, records a `KindRepoAdvanced` mutation and leaves the local records branch at origin's tip".
- **Summary:** by this point the second machine has already adopted the pair in the "adopt" bullet. A plain `Topology.Add` of that slug is refused before the weft probe's answer is used, so it never reaches `probeLiveLocalWeft` or the fast-forward.
- **Evidence:** In `internal/fabricengine/add.go`:
  - The `Add` doc says "An existing local warp branch is refused with ErrBranchExists, unless opts.AdoptLocal adopts it" and "A task worktree path that already exists is refused with *ErrTaskPathTaken".
  - `if warpBranchExists &amp;&amp; !opts.AdoptLocal` refuses at `add.go:349`.
  - The probe answers are read only after the pre-flight refusals: "so a refusal keeps its precedence over a probe's error". The fast-forward at `add.go:565-578` runs after that.
- **What the bullet needs:** a starting state where `Add` gets past those refusals. For example, the second machine's pair worktree is gone while its branches stay, and `Add` runs with `AdoptLocal` (or `AdoptOnly`).

The other added claims hold:
- **Checkout:** `targetWeftTip` fetches only when there is no local branch and origin holds one (`checkout.go:199-218`). The adopt in `switchOrForkWeft` then puts the records worktree on that branch.
- **Records push:** `PushWeft` goes through `PushCoalesced`, and card 2 namespaces it.
- **Owner archives:** `archiveOwnerTip` tags only a tip that origin lacks and pushes it with `pushArchiveTag`, which card 2 namespaces. `readSideOwnerArchives` lists through `remoteTagsUnder` and `remoteHeadTip`, both namespaced in card 3, and creates the local tag.
- **Overview:** it now names exactly the sites the new card 7 bullets reach.

Tool calls:
1. `add.go` spans 216-300, 430-460 and 555-585: my context lacked `Add`'s pre-flight refusals and its fast-forward path.
2. Grep of cards 2 and 3 for `remoteTagsUnder`, `remoteHeadTip`, `pushArchiveTag` and `PushCoalesced`: to confirm the owner-archive and push claims rest on namespaced sites.
3. `add.go` 216-245 plus a grep for the refusal names: to confirm how `Add` treats a slug whose pair or branch already exists.
