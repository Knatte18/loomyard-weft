```yaml
lens: code-correctness
outcome: reported
```

- code-correctness: I traced each function and branch the diff adds or changes, with one concrete case per row.

| Function / branch | Case run | Result |
|---|---|---|
| `gitrepo.Repo.Fetch`, two-repository hub or code store (default refspec) | origin deleted branch `x` | `refs/remotes/origin/x` pruned, live refs kept: correct |
| `gitrepo.Repo.Fetch`, one-repository records store (`+refs/lym/tags/*:refs/tags/*` refspec, `clone.go:829`) | store holds local `set-aside/<slug>/<sha>` tag, never pushed | the tag is deleted by prune: **wrong** (F1) |
| `gitrepo.Repo.Fetch`, non-zero exit | unreachable URL | error names path, exit code and the `fetch --prune` reproduction: correct |
| `judgeAfterCodeTipFetch`, read fails | `readErr` | returns `Verdict{}`, err, no fetch: correct |
| `judgeAfterCodeTipFetch`, non-code-tip verdict | `Held` or clean | returns the first verdict, no fetch: correct |
| `judgeAfterCodeTipFetch`, code tip, fetch fails | `fetchRunForCatchUp` returns `ErrOriginUnreachable` or a `catchUpPair` error | logs at Debug, returns the first verdict, nil: correct by its doc |
| `judgeAfterCodeTipFetch`, code tip, fetch ok, second read fails | read 2 errors | returns that error: correct |
| `judgeAfterCodeTipFetch`, stale `origin/<slug>-weft`, no pair here | fetch prunes the records ref, so `OriginCodeTip` finds no ref and the second judge acts | correct (two-repository hub) |
| `judgeAfterCodeTipFetch`, no pair here, live records branch, code tip pushed since the prime's last code fetch | `fetchRunForCatchUp` fetches only the prime's records store, so `readCodeTipAt` still reads the stale prime code tracking ref | still waits, and the refusal then blames a deleted records branch the fetch just ruled out: imprecise (F2) |
| `runReader.read`, `CodeTipPairAbsent` | `pairHere=false` + `tip.Missing` gives true; `pairHere=true` gives false; tip not missing leaves the zero value | correct |
| `runfsm.Act` copy | `CodeTipPairAbsent` copied into `Verdict` | correct |
| `codeTipRefusal`, `CodeTip=""`, `Machine=""`, `PairAbsent=false` | default reason, "run X again once the code push lands" | correct, unchanged from before |
| `codeTipRefusal`, machine set, `PairAbsent=true` | reason + "; machine m owes it" + the no-pair clause; way forward: pull, then on failure the push | correct |
| `codeTipRefusal`, `Machine=""`, `PairAbsent=true` | way forward: "…run X again; if the refusal stays, run X again once the code push lands" | correct, if repetitive |
| `TestJudgeAfterCodeTipFetch` fake `read` | readErr row with empty `inputs` returns before indexing | correct |

Findings:

**F1 — BLOCKING, design.** Location: `internal/gitrepo/pull.go:42`, `Repo.Fetch` running `fetch --prune`.
In a one-repository hub, every plain fetch of the records store deletes the store's local-only tags, so the commits they alone keep become unreachable.
- Evidence for the refspec: `internal/fabricengine/clone.go:828-829` configures the records store with the explicit fetch refspec `+refs/lym/tags/*:refs/tags/*`.
- Evidence for git's behaviour: `git help fetch`, under `--prune`, says that tags "fetched due to an explicit refspec (either on the command line or in the remote configuration…) … are also subject to pruning".
- Evidence for the lost tags:
  - `snapshotTag` (`internal/fabricengine/snapshotuncommitted.go:100,279-293`) writes a local set-aside tag on `weftRoot` and never pushes it.
  - `RemoveLocal` (`internal/fabricengine/remove.go:219-225,489`) writes a local set-aside tag "in place of the pushed archive tag". Its doc says the commits "stay under the local tag".
  - The `setAsideTag` call sites in `replay.go:518,901` write local set-aside tags.
  - `keepArchiveTag` (`destroy.go:2292,2339`) and the `restore-failed` tag (`destroy.go:2380`) are local too.
  - `pull.go:186` (`weftAnchorStopBeforePull`) and `battencli/catchup.go:224` (`fetchRunForCatchUp`) are two of the records-store fetches that now trigger the prune.
- Fix: prune only the tracking refs, for example `--prune` together with a refspec limited to `refs/remotes/`, or `git remote prune`-style pruning of the heads refspec. Do not prune the records store's `refs/tags/*`. Add a one-repository test asserting that a local set-aside tag survives `Fetch`.

**F2 — LOW, design.** Location: `internal/battencli/arm.go:636` with `internal/battencli/catchup.go:216-230`.
With no pair here, the re-judge fetch refreshes only the prime's records store, never the prime's code store.
`readCodeTipAt` (`internal/fabricengine/originreads.go:144-148`) decides `Missing` from the prime code store's `origin/<branch>`. So a code tip pushed since the prime's last code fetch still waits.
After a successful records fetch, the refusal still says the transition "may be read from a records branch origin has deleted", which the fetch has just ruled out.
The `lym fabric pull` way forward does fetch the code side (`pull.go:604`), so the run is not stranded. But the commit's claim that the act check "fetches the run's branches" holds only for the pair-here case.
Fix: when no pair stands, also fetch the prime's code store, or narrow the commit and summary claims.

Tool calls:
- `clone.go:815-835` plus a grep of the tag-writer call sites plus `remove.go:215-230`. My context held the tag creators but not which store `initRecordsStore` configures with the tag refspec, nor where `snapshotTag`, `setAsideTag` and `keepArchiveTag` are called.
