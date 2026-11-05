---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: handler
    location: PATTERN.md:23
    summary: A cluster entry carries (test) when any member was test-enforced, so clusters with review-only members, such as pair-remote, claim more enforcement than they have, against the documented meaning of the marker
    fix: In `internal/pattern/doc.go`, where it says `(test)` marks a rule a test enforces and is left out for a rule held by review alone, add that on a cluster entry `(test)` means at least one member rule is test-enforced, and that the cluster's background file marks each member rule that is review-only. Then add to each background file of a cluster that carries `(test)` and has a review-only member, a line under that member's `Rule:` line stating it is held by review, starting with the members `Slug remote order` in `pattern/PATTERN-pair-remote.md`, `Wait mark display only` in `pattern/PATTERN-reed-panes.md`, and `Keep-warm cap` in `pattern/PATTERN-daemon-wakeups.md`. The `(test)` markers in `PATTERN.md` stay as they are.
---

The diff makes the change the amended entry asks for.
The overview drops from 15541 to 7532 bytes, about 46% of `MaxOverviewBytes`, and `internal/pattern/check.go` is untouched, so the cap and `HeadroomBytes` stay as they were.
All eight section headings stay, and each cluster sits under the section its subject belongs to.
Every folded entry's old rule line and background text appears verbatim in its cluster's background file, with sub-headings bumped one level and the one self-reference in `PATTERN-lym-files` reworded to point at the sibling section.
A repo-wide grep for each folded ID, excluding `docs/research`, `.lym` and `_lym`, finds none, and every rewritten citation points at a cluster that holds the rule.
The non-pattern diff is comments and string literals only, with no verb or argument changed in any `Errorf` or `Sprintf`.
The commit message and `_lym/landing/summary.md` claims, 64 to 30 entries, ten unchanged entries, 43 files outside `pattern/` and the byte counts, all match the diff.
The directive declares no PATTERN changes, so no declared entry is missing.

### [LOW] Cluster `(test)` marker overclaims enforcement

**Location:** `PATTERN.md:23`, with the same pattern on `PATTERN-lym-files`, `PATTERN-fabric-boundary`, `PATTERN-reed-panes`, `PATTERN-agent-addressing`, `PATTERN-run-records`, `PATTERN-daemon-wakeups`, `PATTERN-stencil-ownership`, `PATTERN-gate-slot` and `PATTERN-gitrepo-client-boundary`.
**Issue:** `internal/pattern/doc.go:71` defines `(test)` as marking a rule a test enforces and leaves it out for a rule held by review alone.
`_lym/landing/summary.md` records the choice that a cluster carries `(test)` when any member did.
A reader of the overview then reads the whole cluster as test-enforced, while members such as slug remote order, the wait mark, the keep-warm cap and the gate fixer were review-only before the fold.
No rule is lost, because each member's own text still states how it is enforced, but the overview line now says more than the guard does.
The checker, `internal/pattern/check.go`, only cuts the marker off the rule text, so no gate catches it.

## Rejected

- Lens reports found no defects, so no lens finding was kept or rejected.
- The cluster rule lines omit some exceptions the old lines carried, such as the heddle rendered-files exception in `PATTERN-lym-files`: rejected, because the amended entry asks for one short rule line per cluster with detail in the background file, and the old line is verbatim there.
- The inconsistent opening sentence of the cluster background files, some beginning "This cluster guards" and some with the rule itself: rejected, no convention in the repository fixes it and no rule is lost.
