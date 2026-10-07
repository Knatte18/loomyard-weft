# tokencount: one flag for the weft repository

`batchifier-dry-run` · Note · enhancement, webster

Left over from card-batching's PR-Gate (2026-10-07); the dry run (`lyx webster validate --batches`) and the peak-context budget have since landed.

`tools/tokencount` has both `-weft` (finished runs) and `-history` (calibration) naming the same weft repository; one flag with one default would do.