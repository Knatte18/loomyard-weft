---
round: 1
focus:
  - "internal/fabricengine tests, every registered template build: does any build call os.Setenv, os.MkdirTemp, git config --global, keep the TB's Context() past the build, call a package-global seam setter or once-only registration (D7 acceptance criteria)?"
  - "Tests that install a process-global seam: is each copy taken before the seam is installed?"
  - "Tests that exercise Add (originreconcile, originreconcileended, originreconcilemoves): do they still add their pairs, and does any per-row slug stay added in the test rather than in a template?"
  - "endedHub and originBoard: does originBoard run on the copy, and does the start state equal the old fixture's by construction (build steps moved unchanged)?"
  - "newAnchorHub, newOriginHub, originPair, endedHub: is each used by at least three sites as D7 requires, and does the diff stay inside the anchor and origin scope rather than the live-state and merge-pair items or other packages?"
  - "Assertions: are the moved tests' assertions unchanged, and does each test keep one tiergate.Require at the lowest fitting tier?"
---

Range is `516b2871..HEAD`: three commits touching five files in `internal/fabricengine`.
