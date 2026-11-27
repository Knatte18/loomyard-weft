```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set: rerunning the searches behind the set-premise findings (F1, F4, F5, F3) and checking the fix covers every member.

Findings left unchecked: F2 (a single assertion, not a set).

**LOW: `_lym/landing/summary.md`, `## Design decisions`, first bullet**
The F1 fix added the `manual` exception to the intro and to `## Changes`, but left "Deriving them leaves the key's tiers unused by the marked scope while every hub runs its marked tests."
That is now false: `markedScope` reads `cfg.Gates[request.gate].Tiers` to decide whether a `manual` tier's marked tests are allowed (`internal/gatecli/only.go:107`). It also contradicts the same file's "a tier declared `manual`: its marked tests run only when the key lists that tier".

**LOW: `_lym/landing/summary.md`, `## Changes`, test bullet**
The bullet still describes only the run of both tiers under a key listing tier0 alone. The F1 and F2 fix also added to that case a refusal for a `manual` tier1, a pass when the key lists it, and a check of tier1's run record.

The other reruns found nothing to report:
- **F1, sites stating the rule:** `only.go`, `doc.go`, `template.yaml` and `PATTERN-test-speed.md` carry the `manual` clause. The `ScopeMarked` comment at `internal/gateslot/tiers.go:131` ("each under its file's tier") says nothing about the listed tiers or refusals, so it is not made false. `cli.go:75` is the same case.
- **F4, the refusal text:** `internal/gatecli/tiers.go:230` is its only site, and no test asserts the old wording.
- **F5, sites implying the removed rule:** no remaining one in `internal`, `cmd`, `pattern` or `docs`.
- **F3, `cli.go` claim:** gone from the summary.

Tool calls (4):
1. grep for `manual` and marked-scope statements across `.go`, `.md` and `.yaml`: my context held only the grep of named files.
2. grep for the F4 refusal wording and tests asserting it: the post-change repo-wide matches were not in context.
3. grep for "marked test" and `upkeep` wording in `internal`, `cmd`, `pattern`, `docs` and `plugins`: sites outside the named files were not in context.
4. `cat` of `summary.md` and `sed` of `gateslot/tiers.go:128-140`: context showed only the edit hunks, and not those comment lines.
