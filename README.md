## overview

This OpenCode configuration is intended to implement test-driven development based on the user's specification `SPEC.md`. Subagent calls are validated against simple formal checks to ensure tests fail and pass as expected. Reusable plain-text artifacts are generated to save context and provide an entrypoint into key decisions made.

Generated markdown documents are distinguished from user-composed markdown files with the extension `.gen.md`.

## installation

To use `zettel-agents` within a git project folder `<project>/`:

```
cd <project>/
git submodule add https://github.com/fleetingfound/zettel-agents.git .opencode/
```

Alternatively, to use `zettel-agents` as the default OpenCode config across projects:

```
git clone https://github.com/fleetingfound/zettel-agents.git ~/.config/opencode/
```

## workflow

1. Create `SPEC.md` describing all core specifications in the project root. This file may be modified during development. It should never be contradicted by the agents.

2. Run `init .` from the project root to initialize the files `Makefile`, `.coreignore` and `AGENTS.md` in the project root.

3. Then run `opencode` from the project root `<project>/`.

Within OpenCode:

4.  Run `/harness` to create and commit the test harness in `./test/`. This implements the commands `make test-unit` and `make test-e2e` which run the unit test suite and end-to-end test suite respectively.

5.  The `planner` agent is then used to resolve key design choices interactively and draft plan documents, one per phase of implementation. To draft an initial plan, it is usually best to use the command `/phase` to invoke the `planner` agent, which will plan a working implementation with a minimum feature set, with each subsequent phase building upon it incrementally and the final phase completing the entire specification, given in `SPEC.md`. Review plans and request revisions as needed.

    Commit with `/commit` once planning is complete.

6.  Run `/flow` to execute the unimplemented plan documents sequentially, with the work of each subagent being verified against simple (and partial) checks. Each subagent will generate its own commit.

7.  As needed, query the `reviewer` agent to generate reviews of the project in `.llm/gen/reviews/`. Commit with `/commit`.
    - The `reviewer` agent may be invoked with `/critique` to provide a general critique of the implementation.

8.  As needed, query the `planner` agent to generate further plans. Commit plans with `/commit` and run `/flow` to implement plans.

## subagent workflows

By default, plans are tagged `#flow/redgreen` which implies that one agent (`@puzzler`) implements failing tests and another agent (`@solver`) implements the plan in order to pass the tests.

- If `@solver` determines that tests are misspecified, then the disputed tests are escalated to `@mediator` to determine whether the issue is in the tests or the implementation. Either `@mediator` repairs the tests or returns the implementation to `@solver` together with an explained verdict.

Other available tags `#flow/pin`, `#flow/refactor`, `#flow/unified` are described in [planner.md](agents/planner.md).

## artifacts

Generated artifacts are saved in the following subfolders:

- `.llm/gen/choices/` - Design chocies and selected designs
- `.llm/gen/plans/` - Plans, one document per phase of implementation
- `.llm/gen/tests/` - Records of test implementation
- `.llm/gen/builds/` - Records of project implementation
- `.llm/gen/issues/` - Issues identified in the implementation or tests
- `.llm/gen/disputes/` - Tests disputed by the `@solver` agent
- `.llm/gen/verdicts/` - Verdicts on disputed tests, according to the `@mediator` subagent
- `.llm/gen/reviews/` - Implementation reviews

Use `.llm/ref/` to save items which agents can refer to but may not edit, such as code examples and documentation for software tools.

## todo

- [ ] Implement [`/flow`](commands/flow.md) as deterministic code which calls LLM subagents instead of as an LLM agent itself

## references

- [Red/Green TDD - Agentic Engineering Patterns](https://simonwillison.net/guides/agentic-engineering-patterns/red-green-tdd/) by Simon Willison
