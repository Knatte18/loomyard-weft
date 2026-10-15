# Fixer report — discussion, round 1

## Changes

All in `_lyx/discussion/decision-record.md`, logged under Review rounds in `_lyx/discussion/support-log.md`.

- F1 fixed as proposed: D1 bounds the work-directory clear by a `.forkaudit-work` marker, deletes only the tool's own intermediates, and exits 1 naming `--work` for a non-empty unmarked directory, the repo root, the hub, `--out`'s directory or an ancestor of any of them, and an `--out` inside it; acceptance criterion added for the unmarked, `--out`-holding and unwritable cases.
- F2 fixed as proposed: Constraints require the `PATTERN-lyxdirs-single-declarer` line in `PATTERN.md` to be amended in the script's commit to admit `tools/forkaudit/forkaudit.sh`'s head variable block; `PATTERN.md` added to Scope In; the `-weft` sentence kept.
- F3 fixed as proposed: D3 names a trace-derived arm `trace:<value>`, only for values matching `[a-z0-9_-]+` that are not marker arm names, and lists any other value once in the samples section as unrecognized; the criterion covers a `raf` value and one holding `|`.
- F4 fixed as proposed, with one addition: D7 searches only `<run>-weft` and `archive/<run>/*`, and also the `origin/<run>-weft` copy, since `git branch -a` in the records store shows remote-tracking records branches; acceptance criterion for two runs sharing a `spawnedAt` added.
- F5 fixed as proposed: D2 states exact project-directory matching and restricts run names to lowercase letters, digits and `-`.
- F6 fixed as proposed: D7 requires `--code` only under `--pack-check` and `--records` under `--pack-check` or `--plan`; the criterion says `--plan` without `--code` succeeds.
- F7 fixed as proposed: D5 sorts by agent id last.

## Disputed

None.

## Deferred

None.
