# Case brief: Darn-Write gate blocked at tier0

## 1. The question the next judge must answer

Is the tier0 guard failure a defect in this run's change, which Darn-Write can fix, or a pre-existing tree problem that needs a code-level decision on the markdown-link check (fix the links, fix the check, or exempt the research data)?

## 2. The facts

- The stop is `blocked` at `Darn-Write`: the gate did not pass after 3 attempts, and the driver reports the chair's gate failed (`/home/knatte/Code/loomyard-LYMHUB/readme-draft-not-ready/.lym/loom/driver.log` lines 16, 21, 25, 29 and 33).
- The envelope `/home/knatte/Code/loomyard-LYMHUB/readme-draft-not-ready/_lym/shed/readme-draft-not-ready/drive-reports/drive-envelope-20261027-154233-cee1.json` names `lym loom resume` as the way forward, which re-spawns Darn-Write with a fresh verify budget.
- Each of the four verify runs (`.lym/verify/verify-1.log` to `verify-4.log`) fails only in tier0: `gate test: tier0 guard discovered 71 passed 69 failed 2 skipped 0` (`verify-4.log`, the tail of the gate output).
- The failing test is `TestEnforcement_MarkdownLinks/repo` (`verify-4.log`, the `--- FAIL` lines), reporting `broken markdown link` on `docs/research/*/data/prompts/step-1-explore-fan.md` lines such as `pattern/PATTERN-cwd-resolution.md` (`verify-4.log` line 63 and the repeats at lines 133, 206, 279, 349, 419, 492, 562).
- The tier0 build tag runs with `GOFLAGS="-trimpath -p=4"`, and the gate output notes that `-trimpath` makes `runtime.Caller` report module-relative paths (`verify-4.log`, the `gate test: tier0 builds with GOFLAGS` line; also driver.log line 17).
- The tier1 and tier2 runs exit 0 (`driver.log` lines 49 and 54). Only tier0 fails.
- The branch diff against `main` touches only `internal/boardengine/` (`doc.go`, `draft.go`, `layer.go`, `layer_test.go`, `render.go`, `render_test.go`); neither the failing test's package nor `docs/research/` appears in it (`git diff --stat main...HEAD`).
- The research directories under `docs/research/` already have commits on `main` (latest touching that path: `752e0035`), so the broken links are present on `main`, not introduced by this branch.
- The working tree is clean (`git status --short` is empty).
- The gate log for the last run is `/home/knatte/Code/loomyard-LYMHUB/_board/.lym/gate/runs/20261027T144210Z-tier0-1803127.log`.

## 3. What I ruled out, and how

- A Darn-Write fault in its own edits: the failing check reads `docs/research/` and the diff does not touch that path or the check's package, so the writer's `internal/boardengine` change does not cause it (checked with `git diff --stat main...HEAD`).
- A transient or environment failure: four independent runs fail identically with the same 69 links (`verify-1.log` through `verify-4.log`), and tier1 and tier2 pass in the same run.
- A retry settling it: the gate already re-prompted three times with the same findings, so a plain `resume` gives a fresh budget but no new input for the writer to act on.
- Whether `TestEnforcement_MarkdownLinks` passes on `main`: not checked. Running `go test` is refused in this session, so I could not confirm the failure is present on an unmodified `main`.

## 4. My hypothesis (not verified)

Hypothesis: the markdown-link check resolves paths in a way that is wrong under `-trimpath` (it uses a `runtime.Caller` path, which the gate note says is module-relative), so every relative link in `docs/research/*/data/prompts/` reports as missing, and the 69 failures are one check misfiring rather than 69 real broken links.
If so, the fix is in the check (locate the repo from the working directory or `go.mod`), not in the research files, and that is a code change for a stronger agent or the run's parent to judge.
A second possibility, to be settled by reading the check: the links really are broken on `main` and the check was always red there.
