# Plan round 2 — a finding surfaced only during the fix phase

The detached-checkout primitives (`CheckoutDetached`/`RestoreBranch` in `gitrepo` and `fabricengine`) losing their only caller turned up only in job B.
The fix for F4 led to a grep for every other "integration bisect" mention, and the review's grep had covered only websterengine's own helpers.
The sequencing rule has no provision for a finding discovered after fixes start.
I appended it to the review as F6, marked as found in job B and recorded before its fix, rather than fixing it silently or leaving it out.
A rubric line on retirement plans would have caught it in job A: sweep the whole repo for the retired mechanism's name and for primitives whose only caller is retired.
