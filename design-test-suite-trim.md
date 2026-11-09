# Coverage maps: build them, keep them fresh, and cut the test suite with them

`test-suite-trim` · Project · enhancement, testkit, gate, dobby

Builds the first full set of coverage maps, settles how they are kept fresh, and uses them to cut the test suite:
cut existing redundant tests, and scope the verify gate to the tests that cover a diff.
