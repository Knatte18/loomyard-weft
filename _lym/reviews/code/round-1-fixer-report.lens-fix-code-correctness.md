```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

The change set is two edits to wave 1 of `contracts/stencils/burler/burler-fork-reading.md:22`. No code, signature or test changed.

- **Parsing:** no Go or test source quotes the edited sentence. The only matches are the stencil itself and measurement records under `docs/research`, which record past runs. The tests that touch this stencil pin the heading, marker placement and marker set, and the diff changes none of them. The stencil's header comment says it declares no marker.
- **Wave 1 after the edits:** it now names a call for each checklist item. File reads at the definitions of the cited symbols, plus the searches for callers, tests, docs and fixtures, callees and other-process paths, failure, restart and resume paths, writers and readers, the board entry and the `pattern/` files. Wave 3 still covers hops the second wave opens.
- **Sentence shape:** the list reads as one grammatical series. The F2 phrase "at the definitions of the symbols it cites" sits inside the file-read item, and a comma separates it from the next item.
- **Tests:** none were added, which fits the review's coverage rejection. The stencil change carries no assertion.

Tool calls made:
1. `grep` of `*.go`, `*.md` and `*.yaml` for the edited phrases, to check that no Go source or test pins the edited sentence. My context held the stencil but not the Go sources.
