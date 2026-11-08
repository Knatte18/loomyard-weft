# Fixing discussion round 1 of redundant-test-finder

The BLOCKING finding F1 rested on a config-layering premise: that a hub's tuned `gates` entry replaces the template's entry whole.
I checked the hub file the finding cited, but not the load path, and built a reinterpretation on that premise.
The fix-claims fork then showed that `yamlengine.FillMissing` fills template keys into each `gates` entry at load, so the premise was false and the change had to be undone.
A review whose premise is a merge rule should cite the merge code, not only the file it merges.
