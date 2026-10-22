---
round: 1
focus:
  - "internal/reportengine/landings.go: does the walk stop at the commit matching the origin-record subject and slug, and does a history without that commit walk as before?"
  - "internal/reportengine/landings.go: does a loom run whose trailer sits above the pair-opening commit still read the same task commit as before, and does a darn run with no trailer fall through to the time join without an exact attempt?"
  - "internal/fabricengine/add.go and origin.go: does pair creation commit the origin record under the exported subject unchanged in text, and is the slug match exact?"
  - "internal/reportengine/landings_test.go: does the regression row cover two darn runs over a parent sync trailer, and does it duplicate an existing row?"
  - "Docs: does reportengine/doc.go state the walk's floor, and do the summary's claims match the diff?"
---

Seed focus from the summary and the diff stat; the diff itself is the subject.
