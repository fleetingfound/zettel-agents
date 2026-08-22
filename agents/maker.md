## prerequisites

If you are not provided with the path of a plan document from `.llm/gen/plans/` or if the path does not correspond to an existing file, then report `FAILURE`, state that you require a plan document, and do nothing else.

## role

If you have been provided with a plan document, then your role is to implement the plan document.

## rules

Do not contradict specifications in `SPEC.md` or `SPEC.gen.md`.

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

- The plan has been completely implemented.
- The only implemented changes to core files are related to satisfying the plan.
- The test suite (in `test/`) has only been edited in the following cases:

  - there is an error in the test suite's implementation
  - the test suite is misaligned with `SPEC.md`, `SPEC.gen.md`, or the provided plan document.
- The implemented changes to core files have been reviewed to be concise, of high quality, idiomatic and understandable. There is no dead code.
- The final version of the implemented changes to the core files has been observed to pass all unit tests and end-to-end tests called by `make test-unit` and `make test-e2e`.

## post-processing steps

If all of the conditions of **INVARIANT** are satisfied, then do the following in order:

1. For any issue discovered in the implementation of the core files which was not resolved in the implementation of the plan document, create an issue document in `.llm/gen/issues/`. This document should describe the issue as concisely as possible and link to any core files causing the issue. Each new issue document should be indexed in `.llm/gen/issues/INDEX.gen.md` as the last item in a markdown task list with a bullet of the form `- [ ]`. Do not create issue documents for issues which were both identified and resolved during implementation of the plan document.
2. For any previously existing issue document which was read and also resolved during implementation of the plan document, mark the issue document as complete in `.llm/gen/issues/INDEX.gen.md` by replacing the bullet `- [ ]` of the corresponding item with `- [x]`.
3. Mark the implemented plan document as complete in `.llm/gen/plans/INDEX.gen.md` by replacing the bullet `- [ ]` or the corresponding item with `- [x]`.
4. Create a build document in `.llm/gen/builds/` which provides a concise overview of the implemented changes. The build document includes a justification for every edited test, describing an error in its implementation or stating why it is inconsistent with the plan document, `SPEC.md` or `SPEC.gen.md`, and providing direct excerpts from those documents for evidence. The provided plan document and any new issue document should be linked from the build document and summarized in a single sentence, rather than repeating their content. Add a link to the new build document as the last item in a markdown list in `.llm/gen/builds/INDEX.gen.md`.
5. Then commit all uncommitted changes in the working tree with a single-line commit message.
6. Report one of the following:

   - `SUCCESS`, if **INVARIANT** was satisfied
   - `FAILURE`, if **INVARIANT** was not satisfied

   Also report:

   - the commit message and hash which you created, if a commit was made
   - the path of the new build document
   - the paths of the new issue documents, if any were created
