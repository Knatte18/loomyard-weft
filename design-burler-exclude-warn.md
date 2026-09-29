Repro: any loom run; every Burler round logs `shedadapters: focus directive names cluster excludes but the template profile has no cluster fan; dropping` (WARN).
Impact: the judge writes an `exclude_lenses` list nothing consumes, and the WARN buries real warnings in traces.

Found in crucible-happy-path round fable-high-r2.

Source: https://github.com/Knatte18/loomyard/issues/278
Preferred fix: stop asking the Burler judge for exclude_lenses (and stop warning) when the template profile has no cluster fan, rather than only downgrading the log level.