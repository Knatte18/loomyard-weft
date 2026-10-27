<!-- This is the guard-fix line of webster's card-gate list in a fork or recovery prompt.
     guardNotice (guardstep.go) reads it through stencilstore, fills its two markers, strips this banner and trailing newline,
     and appends the text after the guard-scan-failure line when the file the scan failed on lies in a package a card targets.
     guard_file is that file's path, and card_path is the pointer of the first card targeting its package.
lyx-stencil: sha256=d42fbea3d6bba6bc3629abde14352b57640e3c6a0f00f31ace091366a19ba39d build=d4824a3d68de466ba3db5015cf6212f7d41126f7 time=2026-10-14T16:27:00Z -->

`{{.guard_file}}` lies in a package the card `{{.card_path}}` targets, so fix that file under that card's commit.
