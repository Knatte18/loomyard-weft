# Card 3 — approval-acted-layout

**Create:**
- `plan:internal/battenshed#approvalActedContent` -> `func approvalActedContent(identity string, historyLen int) string`
- `plan:internal/battenshed#parseApprovalActed` -> `func parseApprovalActed(raw string) (identity string, historyLen int, hasLen bool)`
- `internal/battenshed/approvalacted_test.go`

**Uses:**
- `internal/battenshed#approvalIdentity`

**Intent:**
Add the render/parse pair for the approval-acted marker's two-line layout (D7, D8, D9), as pure helpers in `internal/battenshed/innerrun.go` beside `approvalIdentity`, with no caller until card 4.

`approvalActedContent` returns `identity` (the `approvalIdentity` line, which already ends in a newline) followed by `historyLen` in decimal and a trailing newline.
`parseApprovalActed` splits `raw` after the first newline: everything up to and including it is `identity`, compared byte-for-byte by the caller exactly as the whole file is today.
The remainder, trimmed of surrounding whitespace, parses as a non-negative decimal into `historyLen` with `hasLen` true.
An empty remainder is the old one-line layout (D8) and returns `hasLen` false;
so does a remainder that does not parse (a write torn by a killed process), since the old-layout answer — the re-approve hint — is the conservative one.
A `raw` with no newline at all returns it whole as `identity` with `hasLen` false, so it never matches a real identity line.
The recorded length is carried beside the identity and never folded into it (D9), so identity comparison alone still decides whether an approval is new.

`approvalacted_test.go` is table-driven over `parseApprovalActed`: a round trip through `approvalActedContent` for lengths 0 and a multi-digit value returns the same identity and length with `hasLen` true;
the one-line old layout returns the identity with `hasLen` false;
a non-numeric or negative second line, and a newline-free input, return `hasLen` false with the identity as described above.
