```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness.

- LOW — outputAboveHeader in internal/loomshed/verifygate.go returns "" at the first FAIL<TAB><pkg> line even when that block holds no matching header. A log holding the same package twice, with the failing test in the second block, quotes nothing for it; the old whole-log scan found it. Sibling gap from log concatenation, not a regression of the reviewed premise.

Traced with no defect: F1 (new row fails with the defect in place; bare FAIL<TAB> line guarded), F3 (cut stays rune-aligned and within the bound; row has teeth), F5 (cancelOnWrite reaches the ctx.Done arm deterministically), F2 (priorRecords counted before the stand-in writes; build invocations unaffected; both newCommand callers updated), F4 (assertions pass).

Tool calls: none.
