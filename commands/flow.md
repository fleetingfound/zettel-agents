---
description: Orchestrate test-driven development of plans
agent: orchestrator
---

## precondition

$ARGUMENTS

If you are provided with subpaths of `.llm/gen/plans/`, then these are the target plan documents.

If you are not provided with subpaths of `.llm/gen/plans/`, then your target plan documents are all those plan documents in `.llm/gen/plans/INDEX.gen.md` which remain unchecked (`- [ ]`).

## role

Your role is to orchestrate calls to subagents in a deterministic manner in order to implement the target plan documents.

## rules

Follow all directives precisely. Avoid interpretation or extending your activities beyond explicitly provided directives.

Call only one subagent at a time. The only information which a subagent ever needs is the path of a plan document, with the following exceptions:

- `@mediator` needs the path of a dispute document instead of a plan document
- `@solver` and `@solver.heavy` may receive the path of a verdict document in addition to a plan document when executing a plan for which tests were previously disputed.

At most two dispute documents may be mediated per plan document. If at any point a third `BLOCKED` report occurs for the same plan document, report the discrepancy and do nothing else.

## instructions

Before the first subagent call, run `git log -1 --format=%H -- .llm/gen/runs/` and record its output, so that the verification steps have a baseline for comparison.

Apply the following instructions to each of the plan documents, handling one plan document at a time in the order they appear in `.llm/gen/plans/INDEX.gen.md`.

If in `.llm/gen/plans/INDEX.gen.md` the plan document is tagged `#model/default` or does not have a `#model` tag, do the following:

- If the plan document is tagged `#flow/redgreen` in `.llm/gen/plans/INDEX.gen.md`:

  1. First, pass the path of the plan document to `@puzzler`.
  2. Second, pass the path of the plan document to `@solver`.

- If the plan document is tagged `#flow/pin`:

  1. Pass the path of the plan document to `@pinner`.

- If the plan document is tagged `#flow/refactor`:

  1. First, pass the path of the plan document to `@pinner`.

  2. Second, pass the path of the plan document to `@solver`.

- If the plan document is tagged `#flow/unified` in `.llm/gen/plans/INDEX.gen.md` or if the plan document does not have a `#flow` tag:

  1. Pass the path of the plan document to `@maker`.

If the plan document is instead tagged `#model/big` in `.llm/gen/plans/INDEX.gen.md`, then follow the instructions above, but use:

- `@puzzler.heavy` in place of `@puzzler`
- `@pinner.heavy` in place of `@pinner`
- `@solver.heavy` in place of `@solver`
- `@maker.heavy` in place of `@maker`

After completing a call to any subagent except `@mediator`, `@maker` and `@maker.heavy`, apply the following verification steps.

## verification steps

1. Run `git status --porcelain --ignore-submodules=all`. If this command reveals that there are untracked files or files in the working tree with uncommitted changes, report the discrepancy and do nothing else.
2. If the subagent reports `BLOCKED`, then perform the following substeps:

   - If the subagent was not `@solver` or `@solver.heavy`, or it did not return the path of a dispute document in `.llm/gen/disputes/`, report the discrepancy and do nothing else.
   - Otherwise, pass the path of the dispute document to `@mediator`.
   - If `@mediator` does not report `SUCCESS` together with the path of a verdict document in `.llm/gen/verdicts/`, or `git status --porcelain --ignore-submodules=all` reveals that there are untracked files or files with uncommitted changes after completing the call to `@mediator`, report the discrepancy and do nothing else.
   - Otherwise, run `git log -1 --format=%H -- .llm/gen/runs/`.
   - Pass the paths of the plan document and the verdict document to `@solver`, or to `@solver.heavy` if the plan document is tagged `#model/big`.
   - Once that subagent call completes, repeat the verification steps from step 1.

3. If the subagent does not report `BLOCKED`, then perform the following substeps:

   - Run `git log -1 --format=%H -- .llm/gen/runs/` to determine the last committed change to `.llm/gen/runs/`.
   - If the subagent was `@puzzler` or `@puzzler.heavy`, then also run `git show --format= --unified=0 HEAD -- test/` to determine which tests the latest commit added or edited.
   - Ensure that the following invariant holds with respect to the last called subagent:

     **INVARIANT** (definition):

     - The subagent reports `SUCCESS`.
     - There are no untracked files and no uncommitted changes in the working tree, as indicated by `git status --porcelain --ignore-submodules=all`.
     - Unless the subagent was `@maker` or `@maker.heavy`, the subagent produced a git commit which included a change to `.llm/gen/runs/`, as indicated by a change in the output of `git log -1 --format=%H -- .llm/gen/runs/` since the last time it was called.
     - If the subagent was `@puzzler` or `@puzzler.heavy`, then the field `fail:` in `.llm/gen/runs/test-unit.gen.yaml` or `.llm/gen/runs/test-e2e.gen.yaml` is a non-empty list. In both `.llm/gen/runs/test-unit.gen.yaml` and `.llm/gen/runs/test-e2e.gen.yaml`, all entries of `fail:` are tests which were added or edited by the latest commit.
     - If the subagent was not `@puzzler`, `@puzzler.heavy`, `@maker` or `@maker.heavy`, then no entries appear under `fail:` in either of `.llm/gen/runs/test-unit.gen.yaml` or `.llm/gen/runs/test-e2e.gen.yaml`.

     Only proceed to the next subagent call if every condition of **INVARIANT** is satisfied. Otherwise, report the discrepancy and do nothing else.

## post-processing steps

After all subagent calls have completed without encountering discrepancies, report whether the subagents were successful and summarize their outputs.
