**F1, `rows.go:183` comment, LOW**
- Changed text: "The notice names only what is refused and the scopes it adds; the gate rule itself is stated by another row's notice, see the package doc."
- The proposal swaps a named row for "another row's notice" and adds a "see the package doc" pointer. The code-quality rules ban both: a comment must be correct to a reader who knows no other symbol, and it carries no "see also" pointers.
- The rule-clean version drops the cross-row clause: "The notice names only what is refused and the scopes it adds." The package doc paragraph in `doc.go` already holds the relationship.

**F2, `doc.go:18`: no findings**
- "the steers of the raw go and refuse-raw rows each carry the full rule" is accurate: `rawGoSteer` at `rows.go:161` and `steer` at `rows.go:176` both contain "Gate rule:".
- Naming two rows is load-bearing here, since the point is which rows hold the rule.

Tool calls made: none.
