# Batch 01: gate step naming a package directory that does not exist

Card 2's gate step and the batch gate both list `./internal/planparser_test`.
The external test package lives in `internal/planparser`, so no such directory exists and `lyx gate test` reports `[setup failed]` for it.
Every real package in both steps passed; the step cannot go green without a plan edit, which a fork does not make.
