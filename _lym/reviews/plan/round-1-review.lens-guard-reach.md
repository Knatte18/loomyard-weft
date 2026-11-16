```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: whether each guard, refusal, branch and test row the cards add can be reached.

**Per guard: reachable or not**

Card 1 (`owner-replace-set`)
- Replace selects an open beam whose recorded owner is in the set: reachable. The subtest reaches it with `boxed`, and with `beamer` after `pushFromOther`.
- Replace selects the prime whose recorded owner is in the set: reachable. The fixture's prime owner is `box`.
- An empty string in the set selects nothing: **not reached by its row.** See F2.
- A done entry keeps its owner: reachable. The done beam under `box` is seeded by `pushFromOther`.
- An owner under a name outside the set is left: reachable. The other-name beam is seeded.
- Replace combined with `Slugs` or `Prime` as an added selection: **no caller and no row.** The double-write site is unguarded. See F5.
- The commit subject `board: owner neve replacing box`: reachable. It is asserted.

Card 2 (`machinename-hostname-name`)
- The `HostnameName` error branch is the existing `nameFromHostname` path, already reached by `TestResolveNames`' `UnreadableHostnameNamesItsWayForward` row. No new guard.

Card 3 (`machine-name-verb`)
- `CheckName` refusal: reachable through the row `InvalidNameIsRefused`.
- `FilePath` returning `ok` false: reachable through the row `NoConfigDirIsRefused`.
- `FilePath` error (relative variable), in both set and show: reachable in production, **no row.** See F3.
- `ReadFile` decode failure: reachable through the undecodable-file row.
- `ReadFile` read error, where the file exists but does not read: **unprescribed.** It is folded into "bad file" with no outcome. See F6.
- Hostname error adds no name: reachable through the seam, **no row.** See F7.
- Pull returns a skip, so the hub is checked locally only: reachable through the pull-skip row.
- Pull returns an error, so the hub is checked locally only: reachable, **no row.** See F7.
- `ReadOwners` error, returned naming the hub: reachable from a corrupt `board.json` through `ParseOwners`, **no row.** See F3.
- `hubsToSweep` registry read error: reachable from an unreadable `hubs.json`, **no row.** See F3.
- `hubsToSweep` adds the cwd's hub and dedups: reachable, **no row.** See F7.
- `checkCollision` refusal: reachable through the row `NameOwnedElsewhereIsRefused`. The owner-only-on-origin case needs the fake `Pull` to bring the owner in; the row doesn't say so. See F7.
- `checkCollision`'s hand-write way forward for a name held under a lost file: **its condition cannot be observed, and no row reaches it.** See F4.
- `rewriteFile` writes only on change: reachable through card 4's re-run row.
- Empty `names` entry dropped and reported: reachable through its row.
- Show with a good file, no file or a bad file: **not reachable through the seam.** `show` reads through `machinename.Names`. See F1.

Card 4 (`machine-name-sweep`)
- Write error reported, the next hub swept, exit non-zero: reachable through its row.
- `Lost` entry reported per entry, exit non-zero: reachable through its row. The `lym adopt` addition for a standing owner that is not one of this machine's names: **no row** separates it. See F7.
- Nothing recorded and `HasUnpushed` true, reported as `pushed`: reachable in production and by its row. An earlier commit left unpushed makes `writeOwners` return no paths, and `pushOwnerCommit` then pushes it (ownerset.go:223-224, 131).
- Nothing recorded, plain `nothing to sweep`: **no row.** See F7.
- `HasUnpushed` error, hub reported by its write alone: **no row.** See F7.
- Recoveries printed only after a rename that changed the name: the positive case is reachable; **no row asserts that a same-name re-run omits them.** See F7.

**Findings**

- **F1 — BLOCKING, decision.** Card 3 `show` (03-machine-name-verb.md:61, rows at :87). The show rows cannot reach the good-file, no-file or bad-file state the seam hands in. `machinename.Names` resolves through `machineFile` → `FilePath()` from the process environment, not `Deps.FilePath`, so each row reads the developer's real file. That breaks D8's "every test hands in a file path". `Names` also exposes no `source`. Evidence: internal/machinename/machinename.go:83-85, 119-122.
- **F2 — MEDIUM, consistency.** Card 1 `TestSetOwners` replace subtest (01-owner-replace-set.md:26, :33-36). The `""` in `Replace` never meets an entry whose recorded owner is empty, so the row passes with or without the empty-name guard. After the seed, `beamer` is set to `box` by `pushFromOther`, `boxed` is `box`, and the prime is `box`; no open beam has `owner_machine` empty. Fix: seed an open beam project with no recorded owner and assert it stays unowned. Evidence: internal/boardengine/ownerset_integration_test.go:55-74, 01-owner-replace-set.md:34.
- **F3 — MEDIUM, design.** Card 3 refusal branches with no reaching row:
  - the `FilePath` error from a relative variable, in set (:68) and in show (:63);
  - the `ReadOwners` error naming the hub (:74);
  - the `hubsToSweep` registry read error (:59).

  Each state occurs in production: relative `HOME`, an unparseable `board.json` through `ParseOwners` (ownerset.go:300-307), and an unreadable `hubs.json` through `refuseUnreadable` (hubregistry.go:120-124). None is a raw I/O propagation the refusal spec exempts. Fix: add rows whose fakes return each error.
- **F4 — MEDIUM, consistency.** Card 3 step 6's lost-file way forward (03-machine-name-verb.md:77). The condition "a name this machine held under a file lost to a decode failure" cannot be observed: a decode failure leaves `ReadFile` returning `File{}`, so the verb cannot know which names were held. No row reaches the branch. Fix: state the condition as "the file did not decode" and add a row over an undecodable file whose new name a board records. Evidence: machinename.go:63-65.
- **F5 — LOW, design.** Card 1, "`Replace` combines with `Slugs` and `Prime` as an added selection" (01-owner-replace-set.md:29). No caller sends the combination and no row reaches it. On `Prime: true` with the prime owner in the set, the existing `Prime` branch and the new selection would each append `OwnerWritePrime` to `written`; the same holds for a slug in both `Slugs` and the set. Fix: drop the combination, or prescribe the dedup and add a row. Evidence: ownerset.go:178-184, 221.
- **F6 — LOW, consistency.** Card 3 step 3 (03-machine-name-verb.md:69-70). The step does not separate `ReadFile`'s read error (exists false, err) from its decode error (exists true, err). An implementation reading any error as "does not decode, replaced, names lost" would report lost names over a file it never read. Fix: prescribe that a read error is returned as is. Evidence: machinename.go:55-67.
- **F7 — LOW, design.** Non-refusal branches the cards prescribe with no row:
  - card 3: hostname error adds no name (:71), a pull error leaves the hub checked locally (:73), the cwd hub is added and deduped (:59), and the collision row's owner appears only after the fake `Pull`, which pins pull-before-read for the origin-only acceptance criterion (:87);
  - card 4: plain `nothing to sweep` (:36), a `HasUnpushed` error (:37), the `lym adopt` addition for a `Lost` entry under another name (:34), and recoveries absent on a same-name re-run (:41).

  Fix: add each as a row of `TestName`. Evidence: the cards' row lists at 03-machine-name-verb.md:87-88 and 04-machine-name-sweep.md:46-48.

Tool calls:
1. `sed -n 36,116p internal/boardengine/ownerset_integration_test.go`: read `newOwnerFixture`'s seeded entries and owners, to judge whether the replace subtest reaches an entry with an empty recorded owner. My context had only the grep lines of that file, not the seed.
