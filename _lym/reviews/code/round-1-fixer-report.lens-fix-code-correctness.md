```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1–F4 fix in `internal/selfreportcli` (cli.go and cli_test.go).

No findings.

- **F1 constants.** `bodyFlag` and `bodyFileFlag` replace every literal in `Command` and `resolveBody`. The values are unchanged, so `Changed` and `GetString` behave as before. The `MarkFlagsMutuallyExclusive` row still passes: the gate output shows the `[body body-file]` row passing.
- **F2 padded row.** The input is `" report.md\n"`. `TrimSpace` gives `report.md`, `os.Stat` finds a regular file, and the refusal fires with the trimmed path. Without the `TrimSpace`, `os.Stat(" report.md\n")` fails, the body is filed as text, and exit would be 0. The row would fail with the defect in place.
- **F3 directory row.** `-b "."` stats the temp directory, `IsRegular` is false, and the body is filed as ".". Without `IsRegular`, it would be refused and the row would fail.
- **F3 empty-body row.** `-b ""` is `Changed`, and `existingFileNamedBy("")` returns false. The result is `&""`, and the request carries `"body":""`. The test's `hasBody` switch accepts an empty string, so the row pins empty rather than absent.
- **F4 quoting.** `%q` on the path produces `\"report.md\"` in the JSON-encoded envelope. The test expects exactly the backslash-escaped form, and the gate shows both refusal rows passing.
- **Callers.** `resolveBody`'s signature is unchanged and `runCreate` is unaffected.

No further tool calls made.
