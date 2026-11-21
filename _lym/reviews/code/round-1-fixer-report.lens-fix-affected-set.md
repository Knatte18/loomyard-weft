Task: lens fix-affected-set, checking whether the F1 fix covers every site that carries the door classification.

Unchecked findings: none. F1 is a single wrong classification rather than a set, so I checked only whether another site repeats it.

No findings. The only door classification under `_lym/landing/` is `summary.md:33`, and it now reads `one-way`. The grep found no leftover `two-way` there.

Tool calls:
1. `grep -rniE "two-way|one-way"` over `_lym/landing/`. My context held only `summary.md`, not the other landing files.
