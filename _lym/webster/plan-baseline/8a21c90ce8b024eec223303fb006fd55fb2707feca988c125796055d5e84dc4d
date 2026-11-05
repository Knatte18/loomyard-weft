# Card 8 — report-cost

**Edit:**
- `internal/reportengine#CostOptions`
- `internal/reportengine#CostCell`
- `internal/reportengine#CostReport`
- `internal/reportengine#ComputeCost`
- `internal/reportengine#RenderCostMarkdown`
- `internal/reportengine#usageFiles`
- `internal/reportengine#TestComputeCost`
- `internal/reportcli#costCommand`
- `internal/reportcli#Command`
- `internal/reportcli#TestReportVerbs_RefuseBadFlags`
- `internal/reportcli/doc.go`
- `internal/reportengine/doc.go`
- `//docs/overview.md`

**Uses:**
- `internal/reportengine#ComputeTiming`
- `internal/reportengine#Sources`
- `internal/reportengine#Reader`
- `internal/reportengine#world`
- `internal/reportengine#fakeRepo`
- `internal/reportconfig#LoadConfig`
- `internal/reportcli#timingCommand`
- `internal/reportcli#primeGeometry`
- `internal/shedrun#ParseRunUsage`
- `internal/shedrun#UsageDirRel`
- `internal/shedrun#ReadSeed`

**Intent:** D13, `lym report cost`.
`ComputeCost` takes the landed runs whose archive tag falls in the window (`CostOptions.Since`), whatever their recipe, reads each one's `_lym/usage/` files at the tag's commit (`Records.FilesInDirAtRevision` over `shedrun.UsageDirRel()`), and parses them with `shedrun.ParseRunUsage`, the only reader of those files (`PATTERN-run-usage-record`: never a review round's usage record); a run with no `run.json` is skipped and listed as a finding.
A run is landed when its usage holds `landed.json` (a recipe with a Finalize row, which writes it at landing), or, for a recipe with no Finalize row such as motif and raddle, when its status file at the tag records the run's terminal row done; a cancelled or torn-down run holds neither and is left out.
A run's dollars are the sum of its session records' `cost_usd`; a run with any session of unknown cost enters no dollar figure and enters the hours alone.
A run's active hours are its `ActiveSeconds` from `ComputeTiming` over the same window, joined by tag, for a run the timing report times (tapestry and darn); for any other run they are the union of its session records' `started`–`ended` spans, overlaps counted once, so a motif or raddle cell carries hours too; a run with neither enters the dollars alone.
Runs group into cells by recipe, profile and `stored_record`.
Each cell reports its run count, the mean dollars and mean hours, and, over the runs carrying a `landed.json`, a least-squares fit of a fixed cost plus a cost per 100 changed lines (`lines_added + lines_deleted`) in dollars and in hours, with each fit's correlation.
A run with no landed record, such as a motif or raddle run, enters its cell's means only, so a cell of such runs reports its means and no fit; a fit needs two sized runs with distinct sizes, else it is absent.
A cell with fewer runs than `CostOptions.MinRuns` reads "too few runs" and carries no figure.
`costCommand` is `lym report cost [--since YYYY-MM-DD] [--format json|markdown]`, prime-only like `timingCommand`, its default window the timing report's; it reads `cost_min_runs` with `reportconfig.LoadConfig` into `MinRuns`, and its JSON form, the default, is `{"selection": …, "cells": […], "findings": […]}`, which the raddle agent reads; `Command` adds it.
Each refusal of a bad flag names the flag, its value and the accepted form, as timing's do; `TestReportVerbs_RefuseBadFlags` gains the `cost` rows for `--since` and `--format`.
`TestComputeCost` (tier0, the `world` fake, usage files placed through `tagWith` by `usageFiles`) has rows: a cell of sized runs giving the fixed and per-100-line fit and its correlation in dollars and hours; a cell of motif runs with no landed record (mean dollars and mean hours from their session spans, no fit); a cell of sized runs all of one size (means, no fit); a cell under the minimum ("too few runs"); a run with an unknown-cost session (hours only); a run with no `run.json` (skipped, a finding); a cancelled tapestry run with no `landed.json` (left out); two cells split by the stored-record flag; and a run whose review round usage record disagrees with its `_lym/usage/` files, summed from the usage files.
`docs/overview.md`'s report module line and tree line, the `reportcli` doc and the group's `Long` text (in `Command`) name the cost verb; `reportengine/doc.go` names `ComputeCost`.
The overview's package tree also gains an `internal/reportconfig/` line describing the hub-wide `report.yaml` config, and the report module line's package list names `internal/reportconfig` beside `internal/reportcli` and `internal/reportengine`.

**ImpactSummary:** A new report verb and computation; the existing timing computation is read, not changed, and the CLI group gains one subcommand.
