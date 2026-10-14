---
round: 2
focus:
  - "internal/yamlengine/set.go, SetValues and SetSparse: settle whether the scalar-or-list decision for an open-map path P is made from the template's own value for P before the existing file's value is carried in, so a mapping-declared open map such as labels whose file value is a list still refuses --set labels=x and --set labels=[b] with the file unchanged."
  - "internal/yamlengine/set.go, scalarOrListNode: settle whether a bracketed value with any text after the closing bracket, such as [\"a\"] trailing, is refused with nothing written on both write paths, and whether valid bracketed lists, empty lists and quoted flow lists still parse."
  - "_lyx/landing/summary.md Tests paragraph against internal/yamlengine/layers_test.go and the SetValues tests: settle whether each test row the summary claims exists on the full and the sparse write paths is present on the path it names."
---

Targets the fixes to the round-1 findings, restated by site.
