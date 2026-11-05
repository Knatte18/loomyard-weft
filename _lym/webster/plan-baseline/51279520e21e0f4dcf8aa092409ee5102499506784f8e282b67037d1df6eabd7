# Card 18 — orch-role-landing

**Prosa:**
- `contracts/orchrole/orch-role.md`
- `docs/overview.md`

**Intent:** The beam orch's role text carries a beam's landing workflow (D14).

- `orch-role.md` gains a section after "Making a beam", `## Landing a beam`, for the orch of a beam: the landing is worked at Beam-Await with the operator. When the operator says so the orch runs `lym beam describe <slug>`, `lym beam pr open <slug>`, `lym beam pr update <slug>`, `lym beam pr status <slug>` and `lym beam pr close <slug>`; it changes the description only through the entry's title and brief with `lym board upsert`, then `lym beam describe`, and never edits `summary.md` by hand; it brings the parent in with `lym beam merge-parent <slug>`; on a dobby notice naming landings into the beam it runs `lym beam describe <slug>` then `lym beam pr update <slug>`; it never merges a pull request and never closes one to add work, since work lands into the beam while it stays open, and closes one only when the operator chooses to land without it; it runs `lym beam ready <slug>` only when the operator is satisfied, knowing it is final: no verb takes the beam back to Beam-Await, and a member filed after it never starts. A halted Finalize is followed with `lym beam start <slug>`, which brings the orch back and leaves the dobby down, and `lym loom resume` after the fix the halt names. The section names no `lym beam reopen`.
- `docs/overview.md`'s pawl paragraph names the rows Preflight, Record-Write, Beam-Up, Beam-Await, Finalize, Beam-Down and the orch's landing verbs.
- The verbs' own help texts were written by the cards that added them (12 to 15, 17); no text names `lym beam reopen` after card 6.

This is static prose: the code review reads it, and no test asserts it.

**Commit:** 18: The beam orch's role carries the landing workflow
