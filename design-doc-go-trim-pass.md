# Trim the longest package doc.go files

`doc-go-trim-pass` · Note · enhancement, docs

## What was measured (2026-11-18, `comment-trim-second-pass`)

Production code has 103,762 comment lines against 268,614 code lines, a 27.9 % share.
The package `doc.go` files hold 27,301 of those comment lines; the comment rewrite last week did not shrink them.
Largest: `fabricengine` 2,474, `websterengine` 1,284, `gatecli` 1,273, `reedengine` 1,174, `shuttleengine` 1,075.
By ratio to code: `orchengine` (27 %), `dobbyengine` (16 %), `landingshed` (13 %).
The full measurement is in the change description of that run, records commit `e1d2be8076` in `loomyard-weft`.

## Idea

A narrow pass on `doc.go` only: cut what restates the function docs, keep a package overview.
Start with the largest by absolute size, then by ratio; the operator picks the packages.
`fabricengine` and `landingshed` wait for the runs that change them now.
