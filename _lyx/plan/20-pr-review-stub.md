# Card 20 — pr-review-stub

**Create:**
- `plan:internal/landingshed#PRReview` -> `type PRReview struct`
- `plan:internal/landingshed#NewPRReview` -> `func NewPRReview(name string, enabled bool) *PRReview`
- `plan:internal/landingshed#PRReview.Call` -> `func (p *PRReview) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/landingshed#TestPRReview_Call` -> `func TestPRReview_Call(t *testing.T)`
- `plan:internal/shedrecipe#prReviewEntry` -> `func prReviewEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#TestPRReviewEntry` -> `func TestPRReviewEntry(t *testing.T)`

**Edit:**
- `internal/shedrecipe#registry`
- `internal/shedrecipe_test#coverageGuardAllowedUnreachableEngines`
- `internal/loomengine#Config`
- `internal/loomengine/template.yaml`
- `internal/darnengine#Config`
- `internal/darnengine/template.yaml`

**Extends:**
- `internal/landingshed/doc.go`
- `internal/shedrecipe/doc.go`

**Intent:** Add the PR-Review stub (D7), the seat the human PR review engine will replace: a `PRReview` engine in `internal/landingshed`, registered in `internal/shedrecipe`'s one map (`PATTERN-shed-recipe-registry`).
`PRReview.Call` is Done on every call: with its flag off it logs nothing and acts on nothing; with its flag on it logs one Info that the PR review is not built.
It opens no pull request, calls no GitHub API, reads no file and takes no lock, so it can let nothing through that its absence would not; a cancelled context is an error, as for every landing row.
`prReviewEntry` reads the row's one config key, `pr_review_setting`, a `module.key` setting reference read through `Env.Setting` with the shared bool reader, and refuses every other key.
A missing `pr_review_setting` key is the entry's error naming the key, with the way forward to set it to a `module.key` reference such as `loom.pr_review`; a setting whose value is not a bool is the entry's error naming the setting, with the way forward to set it to true or false through `lyx config <module>` from the prime.
`loom.yaml` gains `pr_review: false` and `darn.yaml` gains `pr_review: false`, each with a line comment naming the stub, and `Config` gains `PRReview` in both modules.
The row is no `Command` row: it takes no operator shell command, and the later review engine replaces it without a command key to retire.
No recipe names the engine until the recipe card adds the PR-Review rows, so `PRReview` joins the coverage guard's allowlist of unreached engines here and leaves it there.

`TestPRReview_Call` pins Done with the flag off and on, the one Info line only when on, and the cancellation error.
`TestPRReviewEntry` pins the setting read for a true, a false and an absent value, a missing `pr_review_setting` key and a non-bool value each with its way forward, and an unknown key.

**ImpactSummary:** One new engine and two additive config keys; the registry gains an entry the coverage guard allowlists until a recipe names it.
