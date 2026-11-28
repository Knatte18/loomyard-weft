```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: I checked that each way forward the fix added can be carried out from the state it names and leads to progress.

**Ways forward checked:**
1. D3's refusal, "set on_done, or use block_gate always".
2. D5's batten-site way forward, `lym board upsert …recipe tapestry`, then `lym batten run`.
3. D5's `lym loom start` way forward, which is D4's jacquard sequence.
4. D4's decision-verb refusals, which name the seed row through `loomrecipe.EngineRow`, as in `lym loom goto --to Record-Seed`.
5. The new acceptance-criterion tests for D3, D4, D5 and D7.

**Findings:**

- **MEDIUM.** D5, the batten-site way forward together with the new done-motif-run clear ("Batten's done-motif-run clear … admits a start whose recipe needs a stored record").
  - Summary: on a slug whose motif run is done and whose entry is set to jacquard with no stored record, the batten way forward leads to another refusal.
  - Evidence: the new clear judges that start as fresh, so the batten refusal fires with the way forward `lym board upsert …"recipe":"tapestry"`. After the upsert, the entry's recipe is tapestry. That recipe is neither raddle nor a recipe that needs a stored record, so `startClearsMotifRun` returns false (`internal/battencli/startguard.go:178`, `recipe != shedrun.RecipeRaddle`). The next `lym batten run` then meets `doneSlugRefusal` (`internal/battencli/arm.go:945-949`), and the stated way forward does not make progress.
  - Possible fixes: for this case, the batten-site way forward names `lym batten cleanup --apply` first, or the clear also admits a tapestry start of an undiscussed entry.

- **LOW (unverified).** D5's `lym loom start` way forward, `lym batten cancel <slug> --remove` from the prime.
  - Summary: for "a start made in a worktree directly", meaning a worktree batten did not make, the cancel may have no batten record to act on.
  - Evidence: `lym batten cancel` help (`internal/battencli/cancelcmd.go`) says it refuses "when nothing is left". I could not establish whether a pair made outside batten counts as something left.

**Tool calls:**
1. Grep of `internal/battencli` for cancel functions. My context held nothing on how `lym batten cancel` treats a slug.
2. Read of `internal/battencli/cancelcmd.go:55-95`, the cancel help text. I needed its refusal conditions for the jacquard sequence.
