```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix change set through the fix-code-correctness lens.

no findings

The three commits (`6df731725`, `6947f06f8`, `d654b4e6b`) change only stencil and spec prose. No function, branch or signature changed and no test was added, so there was no input to step through and no caller to break. The parent's `lym gate test` over `./contracts/...`, `./internal/burlerengine` and `./internal/planparser` reported 0 failed.

Tool calls made: none.
