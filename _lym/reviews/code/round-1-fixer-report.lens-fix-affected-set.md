No findings, both sets checked: `osImportName` has two callers and the fix covers the fslink one the finding named; the only file-keyed `Allowed(rel)` is the one now fixed.
Calls: grep `osImportName`; grep `Allowed(rel)`; grep `import . "os"` in non-test Go files.
