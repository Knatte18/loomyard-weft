```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I checked each changed contract against its callers, tests, docs, specs, stencils and rule documents.

**Per change, the affected set and whether it is covered:**

- **`Deps.LandLocally` doc (`internal/landingshed/deps.go:173-175`), now "tip is the run's own landing commit"**
  - Production caller `pushParent` (`finalize.go:1054`), changed in the diff: covered.
  - Wiring at `landingdeps.go:193`: unchanged signature, so covered.
  - Fake at `finalize_test.go:202`: ignores its argument, so it is unaffected.
  - `envkit.go:99` NilSeams name: unchanged.
  - `Finalize.Call` doc on the already-landed path (`finalize.go:284-285`): not covered. See finding 1.
  - Specs, docs and stencils: none mention the seam.
- **`recordLanding(commit, headErr)` and `pushParent(ctx, landed, headErr, notice)` signatures**
  - Each has one call site, `finalize.go:571-572`, changed in the diff.
  - Tests reach them only through `Finalize.Call`.
  - Covered.
- **Comment at `finalize.go:569`, "The run's own landing commit is the parent's head right after its merge"**
  - Read against the re-entry path that the `Call` doc describes (`finalize.go:280-285`): partly covered. See finding 1.
- **New `writeLedgerLanding` and the slimmer `recordHeldLanding` (`landingdeps.go:946-985`)**
  - Its callers are `recordHeldLanding` and the new test.
  - The `landLocally` doc was updated in the diff.
  - The `pushledger.Landing.Tip` doc, "the parent tip the landing made" (`pushledger/ledger.go:77`), stays true.
  - `runpusher.RecordLanding` is unchanged.
  - Covered.
- **The branch read moved under the push lock and after the status commit**
  - No doc or rule document describes the order.
  - PATTERN-pair-remote's "While a landing entry stands, no push…" (`pattern/PATTERN-pair-remote.md:34`) is unchanged and needs no edit.
  - The failure-state question is code-correctness's.
- **PATTERN and rule documents**
  - The directive declares no PATTERN changes, and the diff touches no `PATTERN.md` or `pattern/` file. Nothing is owed.
  - PATTERN-landing (`pattern/PATTERN-landing.md:9-10`) still holds.
- **Commit messages in the range (`5effc4922`, `56f477eaf`, `be85ce0a7`, `e38f42903`)**
  - No `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailers.
  - None is needed: no test at the base is loosened, and no bug fix is asked for.
  - Covered.

**Findings:**

1. **MEDIUM, consistency.** `internal/landingshed/deps.go:174` and `internal/landingshed/finalize.go:569`.
   - **Summary:** the new seam contract says "tip is the run's own landing commit", and the new comment says the post-merge head is the run's own landing commit.
     The unchanged `Finalize.Call` doc says that on a re-run over an already-landed parent, "the landing is recorded again with the parent's head".
     After later runs have landed on top, that head is not the run's own landing commit.
     So either the new contract is false on that path, or the `Call` doc is now stale; the diff leaves the two contradicting each other.
   - **Evidence:** `internal/landingshed/finalize.go:280-285` (`Call` doc: an empty squash commits nothing, "the landing is recorded again with the parent's head").
     `finalize.go:570` reads `parentHandle.HeadSHA()` after a merge that may commit nothing.
     The new claims are at `deps.go:174` and `finalize.go:569`.
     Whether the code itself is wrong on this path is code-correctness's call. This finding is the doc set the diff leaves contradicting itself.

Tool calls:
1. A grep of `_lym/specs`, `docs` and the stencil directories for ledger-landing and `LandLocally` wording: the manifest's earlier `.md` search excluded `_lym` and skipped the stencils.
2. A grep of landing, runpusher and pushledger Go doc comments for "parent head" wording, plus the `envkit` seam list: the manifest held no reader of the old "parent head" phrasing outside the diff.
3. A read of `finalize.go:275-300` (the `Call` doc) and a grep of `doc.go`: the second grep pointed at line 285, which the manifest had not read.
