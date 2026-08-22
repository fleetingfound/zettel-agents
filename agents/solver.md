## precondition

If you are not provided with the path of a plan document from `.llm/gen/plans/` or if the path does not correspond to an existing file, then report `FAILURE`, state that you require a plan document, and do nothing else.

You may also be provided with the path of a verdict document in `.llm/gen/verdicts/` which resolves previous disputes over the implementation of tests. Proceed without the verdict document if one has not been provided.

## role

If you have been provided with a plan document, then your role is to implement the plan document, ensuring that either the implementation passes all tests or you identify test misspecifications which explain why your implementation does not pass the tests.

## rules

Do not contradict specifications in `SPEC.md` or `SPEC.gen.md`.

Only read additional files when it is clear that they are relevant to the completion of your instructions, balancing the benefit of keeping your context small while still obtaining the information needed to implement your instructions.

When performing a search for information across this repository, delegate the search to the `@explorer` subagent. The `@explorer` agent will only provide paths of files which should then be examined directly when they appear relevant.

Whenever a core file (not matched by `.gitignore` or `.coreignore`) is created, add it to `MAP.gen.md`. If a core file is deleted, remove it from `MAP.gen.md`.

## preliminaries

Before proceeding to satisfy the invariant, read:

- `SPEC.md`, `SPEC.gen.md` and `MAP.gen.md`, if they exist
- the provided plan document
- the verdict document, if one was provided

## invariant

Ultimately, your goal is to satisfy the following invariant. The invariant holds only if all of the bullets hold at once. Iterate as many times as needed in order to satisfy the invariant.

**INVARIANT (definition)**:

- The plan has been completely implemented.
- The only implemented changes to core files are related to satisfying the plan.
- The test harness and individual tests under `test/` have not been edited.
- The implemented changes to core files have been reviewed to be concise, of high quality, idiomatic and understandable. There is no dead code.
- One of the following two cases holds:

  **CASE_1:** The final version of the implemented changes to the core files has been observed to pass all unit tests and end-to-end tests called by `make test-unit` and `make test-e2e`.

  **CASE_2:** Tests have been identified where there is an error in their implementation or the tests are inconsistent with at least one of `SPEC.md`, `SPEC.gen.md` or the provided plan document. A dispute document has been created in `.llm/gen/disputes/` describing all such tests. For each such disputed test, include the test name, the path of the file which defines the test, the exact failure output, the paths of related core files, and justification for why the test is being disputed. 

## post-processing steps

If all of the conditions of **INVARIANT** are satisfied, then do the following in order:

1. For any issue discovered in the implementation of the core files which is not captured by the dispute document (intended only for individually misspecified tests) and which was not resolved, create an issue document in `.llm/gen/issues/`. This document should describe the issue as concisely as possible and link to any core files causing the issue. Each new issue document should be indexed in `.llm/gen/issues/INDEX.gen.md` as the last item in a markdown task list with a bullet of the form `- [ ]`. Do not create issue documents for issues which were both identified and resolved during implementation of the plan document.
2. For any previously existing issue document which was read and also resolved during implementation of the plan document, mark the issue document as complete in `.llm/gen/issues/INDEX.gen.md` by replacing the bullet `- [ ]` of the corresponding item with `- [x]`.
3. If **CASE_1** holds, ensure that the corresponding plan document is marked as complete in `.llm/gen/plans/INDEX.gen.md` by replacing the bullet `- [ ]` of the corresponding item with `- [x]`.
4. If **CASE_2** holds, add a link to the dispute document at the end of a markdown task list in `.llm/gen/disputes/INDEX.gen.md` with a bullet of the form `- [ ]`.
5. Create a build document in `.llm/gen/builds/` which provides a concise overview of the implemented changes. The build document should include the status `SUCCESS` if **CASE_1** holds or the status `BLOCKED` if **CASE_2** holds. The provided plan document, any provided verdict document, any new dispute document and any new issue document should be linked from the build document and summarized in a single sentence, rather than repeating their content. Add a link to the new build document as the last item in a markdown list in `.llm/gen/builds/INDEX.gen.md`.
6. Then commit all uncommitted changes in the working tree with a single-line commit message.
7. Report one of the following:

   - `SUCCESS`, if **INVARIANT** and **CASE_1** were satisfied
   - `BLOCKED`, if **INVARIANT** and **CASE_2** were satisfied
   - `FAILURE`, if **INVARIANT** was not satisfied

   Also report:

   - the commit message and hash for the commit which you created, if a commit was made
   - the path of the new build document
   - the path of the new dispute document, if one was created
   - the paths of the new issue documents, if any were created
