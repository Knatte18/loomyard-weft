```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff forces outside the lines it touches.

**Per change, the set I found:**

| Change | Set found | Covered? |
|---|---|---|
| `Config.TrailerText` / `EngineTrailers` replaced by `Credit*`, and `TrailerBlock` replaced by `CreditBlock` | `finalize.go`, `prreview.go`, `finalize_pr.go`, `deps.go`, `envkit.go` (`nilLegal`), and the tests (`config_test`, `finalize*_test`, `prreview*_test`, `publish_test`) | Covered. A repo-wide grep finds no remaining `TrailerBlock`, `TrailerText`, `EngineTrailers` or `taskTrailer` outside retired-key lists and test rows that exercise the retired keys. |
| `Lym-Task` line and `taskTrailer` removed | The only remaining `Lym-Task` hit is the new negative assertion in `finalize_test.go:312`. `reportengine` (`landings.go:153`) and `fabricengine` read `TaskCommitOf` and the origin-record subject (the weft-side `Warp-SHA` trailer), not this line. | Covered; no consumer of the removed line. |
| landing.yaml keys: four new, three retired | `template.yaml`, `ConfigSpec` (all four cross-pair, `credit_engines` the only open map), `RetiredKeys`, `MigrateConfig`, `doc.go:204`, `docs/overview.md:598`, `config_test` | Covered. |
| PR body and merge message now carry the credit block | `prreview.go` (Create and Edit both use `body`), `finalize.go:276`, `doc.go:7,199-203,222`, `loom-final-summary-spec.md`, `prreview.go` doc comments | Covered. |
| `summaryparser` finding detail | `description.go:68` updated; the tests assert the check id, not the detail text | Covered. |
| Describe stencil rule | `landing-template-describe.md:51` now says no `Co-Authored-By` and no credit lines, which matches the board entry. No rule is dropped. | Covered. |
| `PATTERN.md` / `pattern/` | `PATTERN-config-layers` (retired keys go through `RetiredKeys`, only `configengine` reads layers), `PATTERN-sparse-hub-base`, `PATTERN-tolerant-records`. The diff follows the first. No declared PATTERN changes exist and no `Pattern-Departure:` trailers are used. | Covered. |

**Findings (both outside the lines the diff touched):**

1. **LOW / consistency** — `internal/summaryparser/summary.go:87-94`. The `LandingMessage` doc comment and its `trailers` parameter still describe a trailers block, but the argument is now the plain-line credit block. Finalize is the only non-test caller, at `finalize.go:195`. The doc even says "trailers verbatim", which the new `doc.go:199` contradicts ("never as git trailers"). Evidence: `summary.go:87-90`.
   Fix: rename the parameter to `credit` and reword the comment to "then a blank line and credit when credit is non-empty". `description_test.go:83-85` can keep its row, or be renamed.

2. **LOW / consistency** — `internal/loomcli/prcontinue.go:3`. The comment says Finalize "lands the run locally under its own gate and trailer", but Finalize no longer appends a trailer. `finalize.go` already changed the parallel phrase to "gate and credit block".
   Fix: change "gate and trailer" to "gate and credit block".

Nothing else stale. The hits in `docs/research` and `docs/benchmarks` are historical reports and the `fabricengine` `Warp-SHA` hits are out of scope, so I did not flag them.

**Tool calls:**
1. `grep -rniE 'trailer|Lym-Task|engine line|co-author'` over md, yaml and go, to find stale prose outside the diff.
2. `grep` for `Lym-Task` and the task-commit trailer, plus reads of `reportengine/landings.go`, `reportengine/doc.go` and `loomcli/prcontinue.go`, to check whether anything depends on the removed line and to see the stale `prcontinue.go` comment.
