## role

Your role is to assist in the implementation of the current project.

## rules

Do not contradict specifications in `SPEC.md` or `SPEC.gen.md`.

Only read additional files when it is clear that they are relevant to the completion of your instructions, balancing the benefit of keeping your context small while still obtaining the information needed to implement your instructions.

When performing a search for information across this repository, delegate the search to the `@explorer` subagent. The `@explorer` agent will only provide paths of files which should then be examined directly when they appear relevant.

Whenever a core file (not matched by `.gitignore` or `.coreignore`) is created, add it to `MAP.gen.md`. If a core file is deleted, remove it from `MAP.gen.md`.

You may edit the test suite only in the following cases:

- there is an error in the test suite's implementation
- the test is misaligned with `SPEC.md`, `SPEC.gen.md`, or the provided plan document.

For any issue discovered during implementation of the core files, create an issue document in `.llm/gen/issues/` which describes the issue as concisely as possible and links to any core files causing the issue. Each new issue document should be indexed in `.llm/gen/issues/INDEX.gen.md` as the last item in a markdown task list with a bullet of the form `- [ ]`.

For any issue document from `.llm/gen/issues/` which you resolve, mark the corresponding item in `.llm/gen/issues/INDEX.gen.md` as complete by replacing `- [ ]` with `- [x]`.

After instructions are completed, create a build document in `.llm/gen/builds/` so that it provides a concise overview of the implemented changes, or update the existing build document if one was already created in this conversation. Every edited test requires justification, describing an error in its implementation or stating why it is inconsistent with the plan document, `SPEC.md` or `SPEC.gen.md`, and providing direct excerpts from those documents for evidence. The provided plan document and any new issue document should be linked from the build document and summarized in a single sentence, rather than repeating their content. Add a link to the new build document as the last item in a markdown list in `.llm/gen/builds/INDEX.gen.md`.

## preliminaries

Before proceeding with instructions, read:

- `SPEC.md`, `SPEC.gen.md` and `MAP.gen.md`, if they exist
- the plan document and any files it references, if provided with a plan document
