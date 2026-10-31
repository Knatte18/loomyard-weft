```yaml
lens: fix-affected-set
outcome: reported
```

The test in `description_test.go` still names its field `trailers`, which is a missed member of F3's set.

Findings:

- **LOW** — `internal/summaryparser/description_test.go:91-92`: `TestLandingMessage` still passes a field named `tt.trailers` to `LandingMessage(credit)` and prints it in the error message. F3 renamed the parameter in `summary.go:90`, and this is the same stale term in a test that calls it. I did not read the table's definition; the field name at lines 91-92 is the evidence. The fix is to rename the field to `credit`.

No other stale "trailer" wording turned up. The `trailer` hits in `landingshed/doc.go:199,204` and `config.go:156,161` are intentional, describing "never git trailers" and the retired `trailer_text` and `engine_trailers` keys. The remaining hits are in the fabric and webster git-trailer code, which has nothing to do with the credit block. For F4, `prcontinue.go` was the only "gate and trailer" phrasing. For F1 and F2, each is a single site, and `TestPRReview_Call` is the only PR-Review test that builds on the fixture. I left nothing unchecked.

Calls made:
1. Grepped `trailer` across Go files in `internal`, to rerun F3's and F4's searches for stale wording.
2. Grepped `LandingMessage` across `.go` and `.md` files, to find every caller and doc reference for F3.
