# Card 8 — PATTERN-verified-tree names the pause hold, and its tripwire checks it

**Edit:**
- `cmd/lyx#TestVerifiedTree_PlanVerifySitesCallVerify`
- `PATTERN.md`
- `pattern/PATTERN-verified-tree.md`
- `pattern/PATTERN-sparse-hub-base.md`
- `pattern/PATTERN-config-layers.md`
- `pattern/PATTERN-mutation-record.md`
- `pattern/PATTERN-friction-capture.md`
- `pattern/PATTERN-testkit.md`
- `pattern/PATTERN-hubforge-fixtures.md`
- `pattern/PATTERN-gogit-read-helper.md`
- `pattern/PATTERN-cwd-resolution.md`

**Uses:**
- `cmd/lyx#planVerifySites`

**Intent:**
The declared `PATTERN-verified-tree` change, with its tripwire, and the byte room the two declared entries need.

`PATTERN.md`'s `PATTERN-verified-tree` line becomes, verbatim:

```markdown
- `PATTERN-verified-tree` — Running a verify command: only via `verifytree.Verify`, told its scope's pause hold. (test) — [background](pattern/PATTERN-verified-tree.md)
```

`pattern/PATTERN-verified-tree.md` gains a bullet: every site tells `Site.Hold`, built by `verifytree.NewPauseHold` from its wiring's pause geometry, and the Merge-In, Finalize and Publish site's hold reads hub entries only while its run holds the parent lock; `cardverify` and battencli's `verifyrun` rows tell none, being no verify site.
Its Enforcement section adds that each site file also names a `Hold` it passes.

`TestVerifiedTree_PlanVerifySitesCallVerify` also fails a site in `planVerifySites` whose code lines do not contain `Hold:` or `.Hold =`, naming the entry and the site, so a site that drops its hold trips it; its doc comment says so.

The overview stands at 15703 bytes at the merge base, past `pattern.HeadroomBytes` (15564), so `patternlint`'s headroom lint, which the Webster-Burler-Fix and darn verify gates run, fails any growth of it on this branch, and `patternlint.WayForward` is to cut bytes by moving detail into the entries' background files.
This card and card 9 together grow the overview by 292 bytes (this card's 29, card 9's new 262-byte line and its newline), so this card cuts these trailing clauses from their entry lines, 313 bytes in all, and adds each clause's rule to that entry's background file where the file does not already state it:

- `PATTERN-sparse-hub-base`: `; only `hubreconcile`'s one-time walk strip drops a unit the template holds`
- `PATTERN-config-layers`: `, machine layer included`
- `PATTERN-mutation-record`: `; its result embeds `MutationRecord``
- `PATTERN-friction-capture`: `, reflected at the run's end, archived once covered`
- `PATTERN-testkit`: `, never duplicated`
- `PATTERN-hubforge-fixtures`: `, never hand-assembled or wrapped`
- `PATTERN-gogit-read-helper`: `, sole caller of `goGit()``
- `PATTERN-cwd-resolution`: `; nothing else calls `os.Getwd` or `git rev-parse``

Each entry keeps its rule: the cut clause moves word for word into its background, and every entry line keeps its `(test)` marker and link.
After card 9 the overview ends 21 bytes shorter than at the merge base.

**ImpactSummary:** the overview's verified-tree line and eight entry lines shortened with their clauses moved to their backgrounds; one tripwire gains a check over the four existing site files, all of which carry a hold after cards 5 to 7.
