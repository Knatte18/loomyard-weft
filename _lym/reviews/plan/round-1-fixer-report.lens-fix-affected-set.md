```yaml
lens: fix-affected-set
outcome: reported
```

I re-ran the search for each finding whose premise is a set: F1, F3 and F4.

Unchecked list: none. F2, F5 and F6 are not sets.

- **F4: covered.** The search finds three sites for the parent-branch reroute clause: `internal/loomcli/wiring.go:955`, `internal/loomcli/wiring_test.go:1052` and `internal/loomcli/wiring_test.go:1099`. The fix lists all three, and its Tests line names both fragments.
- **F3: covered.** The search finds `internal/battenshed/deps.go:293` and `internal/battenshed/innerrun_binaryrestep.go:53`, and the fix lists and documents both.
  It also hits `internal/sheddriver/doc.go:30`, `internal/sheddriver/driverules.go:49` and `internal/sheddriver/driverules.go:219`. Those describe the envelope's `hand_back` mark, which an escalate halt never gets, so they stay true and are not in the set.
  The other hit, `internal/landingshed/doc.go:167`, is about lock halts and is out of scope.
- **F1: one member missed.**
  - LOW. Changed text: card 1, the F2 fix sentence ("`Finalize.Call`'s doc rewrites its merge sentence too…; `Finalize.mergeIntoTask`'s doc drops the loom abort") and the F1 Docs sentence.
  - Summary: `Finalize.mergeParent`'s doc still names a large merge as a reroute cause, and no card line rewrites it.
  - Evidence: `internal/landingshed/finalize.go:838-843` reads "The merge is large when it needed the conflict session…", "A large merge keeps the merge commit and reroutes, with no gate run and no fixer; a darn Finalize first writes the overlap note…" and "With Deps.LargeMergeLands a large merge is neither rerouted nor noted…".
  - Card 1's Intent changes `mergeParent`'s behaviour (it drops the aborted-conflict reroute, the overlap-note write and the large-merge `switch`) but never says its doc changes. The doc names `Deps.LargeMergeLands`, which card 1 deletes.
  - The other hits in this search are already handled:
    - `internal/landingshed/doc.go:45-46` is the Finalize merge paragraph, which card 1 already covers.
    - `internal/landingshed/doc.go:120` is the fixed HandsBack paragraph.
    - `internal/landingshed/deps.go:162` and `internal/loomcli/landingdeps.go:265` are deleted with their fields "and their comments".
    - `internal/landingshed/overlapnote.go:1` is in a deleted file.
    - `internal/loomcli/doc.go:17` and `docs/overview.md:568` each already have a Docs line.

Tool calls:
1. Searched for the parent-branch reroute clause across the repo, because my context held only the two test line numbers, not a full sweep (F4).
2. Searched for "landing halt only" phrasing, because my context held only the two sites the review named (F3).
3. Searched for large-merge or loom-conflict reroute wording across code and docs, because my context held only `internal/landingshed/doc.go:120` (F1).
4. Read `internal/landingshed/finalize.go:836-846`, because my context lacked the doc comment that hit 3 pointed to.
