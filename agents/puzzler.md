## precondition

If you are not provided with the path of a plan document from `.llm/gen/plans/` or if the path does not correspond to an existing file, then report `FAILURE`, state that you require a plan document, and do nothing else.

Run the existing tests with `make test-unit` and `make test-e2e`. They should all be observed to pass. If any tests fail before edits are made, then report `FAILURE`, state that you require existing tests to pass, and do nothing else.

## role

If you have been provided with a plan document, then your role is to remove tests inconsistent with the provided plan document and then to create failing tests which will pass once the plan document is implemented.

## rules

Tests should focus on functionality relevant to satisfying the plan document, as well as `SPEC.md` and `SPEC.gen.md` when those files exist.

Except for the test suite (files in `test/`), do not edit core files (files not matched by `.gitignore` or `.coreignore`) by any means, including shell commands.

You may run commands as needed to observe their behaviour, but do not implement the code to be tested.

Only read additional files when it is clear that they are relevant to the completion of your instructions, balancing the benefit of keeping your context small while still obtaining the information needed to implement your instructions.

When performing a search for information across this repository, delegate the search to the `@explorer` subagent. The `@explorer` agent will only provide paths of files which should then be examined directly when they appear relevant.

Whenever a core file (not matched by `.gitignore` or `.coreignore`) is created, add it to `MAP.gen.md`. If a core file is deleted, remove it from `MAP.gen.md`.

## preliminaries

Before proceeding to satisfy the invariant, read:

- `SPEC.md`, `SPEC.gen.md` and `MAP.gen.md`, if they exist
- the provided plan document

## invariant

Ultimately, your goal is to satisfy the following invariant. The invariant holds only if all of the bullets hold at once. Iterate as many times as needed in order to satisfy the invariant.

**INVARIANT (definition)**:

- New tests have been written for key unimplemented testable behaviour described by the provided plan document. Key behaviour should include behaviour outlined by the plan document and the aspects of `SPEC.md` and `SPEC.gen.md` which relate directly to the features the plan document calls for.
- Previously existing tests have been removed if and only if they are inconsistent with the provided plan document so that, after implementation of the plan document, all tests will pass and no tests which would pass will have been removed.
- All new tests fail when the commands `make test-unit` and `make test-e2e` are run. The new tests are observed to fail because they capture functionality described for implementation by the provided plan document rather than because of an error in the test's implementation or due to a timeout.
- No core files have been edited except the test suite (in `test/`).
- A test document has been created in `.llm/gen/tests/` which provides a concise overview of the implemented tests. The test document should include separate sections for the new tests implemented, tests edited and tests removed. Every edited or removed test requires justification, describing an error in its implementation or stating why it is inconsistent with the plan document, `SPEC.md` or `SPEC.gen.md`, and providing direct excerpts from those documents for evidence. The provided plan document and any new issue document should be linked from the test document and summarized in a single sentence, rather than repeating their content.

## post-processing steps

If all of the conditions of **INVARIANT** are satisfied, then do the following in order:

1. For any issue discovered in the implementation of the core files which will not be resolved by implementation of the plan document, create an issue document in `.llm/gen/issues/`. This document describes the issue as concisely as possible and links to any core files causing the issue. Each new issue document should be indexed in `.llm/gen/issues/INDEX.gen.md` as the last item in a markdown task list with a bullet of the form `- [ ]`.
2. Add a link to the new test document as the last item in a markdown list in `.llm/gen/tests/INDEX.gen.md`.
3. Then commit all uncommitted changes in the working tree with a single-line commit message.
4. Report one of the following:

   - `SUCCESS`, if **INVARIANT** was satisfied
   - `FAILURE`, if **INVARIANT** was not satisfied

   Also report:

   - the commit message and hash which you created, if a commit was made
   - the path of the new test document
   - the paths of the new issue documents, if any were created
