## role

Your role is to establish the project's test harness.

## rules

Do not contradict specifications in `SPEC.md` or `SPEC.gen.md`.

Only read additional files when it is clear that they are relevant to the completion of your instructions, balancing the benefit of keeping your context small while still obtaining the information needed to implement your instructions.

When performing a search for information across this repository, delegate the search to the `@explorer` subagent. The `@explorer` agent will only provide paths of files which should then be examined directly when they appear relevant.

Whenever a core file (not matched by `.gitignore` or `.coreignore`) is created, add it to `MAP.gen.md`. If a core file is deleted, remove it from `MAP.gen.md`.

## preliminaries

Before proceeding to satisfy the invariant, read:

- `SPEC.md`, `SPEC.gen.md` and `MAP.gen.md`, if they exist

## invariant

Ultimately, your goal is to satisfy the following invariant. The invariant holds only if all of the bullets hold at once. Iterate as many times as needed in order to satisfy the invariant.

**INVARIANT (definition)**:

- The harness has been completely implemented.

- The only implemented changes to core files (files not matched by `.gitignore` or `.coreignore`) are related to implementation of the test harness, which is defined by `Makefile`, files with extension `.mk`, and files in `test/`.

- The only tests which have been implemented are tests intended to test aspects of the test harness.

- The test harness is implemented to provide the following commands:

  - `make test-unit` for calling unit tests
  - `make test-e2e` for calling end-to-end tests
  - `make test-canary` for calling canary tests to test the harness during harness development
  - `make test-timeout` for calling timeout canary tests to test the harness's timeout enforcement during harness development

- Every other aspect of the test harness described in `SPEC.md` or `SPEC.gen.md` has been implemented.

- The test commands must support a timeout specified by the shell variable `GLOBAL_TEST_TIMEOUT` which is applied to the test suite as a whole and a timeout specified by the shell variable `INDIVIDUAL_TEST_TIMEOUT` which is applied to individual tests. These values enable the timeouts of a test suite to be adjustable via invocations such as `GLOBAL_TEST_TIMEOUT=1200 INDIVIDUAL_TEST_TIMEOUT=60 make test-unit`, so that timeouts can be adjusted without editing files in `test/`. When those variables are not explicitly set, default timeout values are used instead.

- Each of the commands `make test-unit`, `make test-e2e`, `make test-canary` and `make test-timeout` should exit 0 if and only if every test in its respective test suite is passed without reaching the timeout. Each command must exit nonzero when any test fails or if a timeout is reached.

- Each of the commands `make test-unit` and `make test-e2e` should also overwrite the corresponding run record `.llm/gen/runs/test-unit.gen.yaml` or `.llm/gen/runs/test-e2e.gen.yaml`, which takes the following form:

  ```
  target: test-unit
  started: '2026-07-30T14:21:03Z'
  exit: 1
  totals: { total: 24, pass: 21, fail: 3, skip: 0 }
  fail:
   - 'test/unit/rooms.bats :: create room persists to disk'
  ```

  where

  - `started:` gives the UTC time at which the run began, single quoted
  - `exit:` gives the command's exit status or `timeout` if a timeout was reached
  - `total:` gives the total number of tests
  - `pass:` gives the number of passing tests
  - `fail:` gives the number of failing tests
  - `skip:` gives the number of skipped tests
  - `fail:` consists of entries which each name a failing test and must be single quoted with additional single quotes escaped

  Writing the yaml record must not alter the exit status of the command.

- Canary tests are implemented in `test/canary/` which test something false. These are observed to fail when invoked with the command `make test-canary`.

- Timeout canary tests are implemented in `test/timeout/` which test non-terminating commands.

  - When `make test-timeout` is invoked with sufficiently small values of `INDIVIDUAL_TEST_TIMEOUT` and a sufficiently large value of `GLOBAL_TEST_TIMEOUT`, these are observed to fail due to test-level timeouts and `make test-timeout` returns a non-zero exit error.
  - When `make test-timeout` is invoked with sufficiently small values of `GLOBAL_TEST_TIMEOUT` and a sufficiently large value of `INDIVIDUAL_TEST_TIMEOUT`, these are observed to fail due to suite-level timeouts and `make test-timeout` returns a non-zero exit error.

- Smoke tests implemented in both `test/unit/` and `test/e2e/` must be observed to pass and the correct records in `.llm/gen/runs/` created when run respectively with the commands `make test-unit` and `make test-e2e`.

- The implemented changes to core files have been reviewed to be concise, of high quality, idiomatic and understandable.

- A build document has been created in `.llm/gen/builds/` which provides a concise overview of the implemented harness, including any issues encountered and how they were resolved, as well as any important design decisions which were made. The build document includes the exit status of each test run with `make test-unit`, `make test-e2e`, `make test-canary`, and `make test-timeout`. A link to the new build document has been added as the last item in a markdown list in `.llm/gen/builds/INDEX.gen.md`.

## post-processing steps

If all of the conditions of **INVARIANT** are satisfied, then commit all uncommitted changes in the working tree with a single-line commit message.

Then do not edit any files further.

Report one of the following:

- `SUCCESS`, if **INVARIANT** was satisfied
- `FAILURE`, if **INVARIANT** was not satisfied

Also report:

- the commit message and hash which you created, if a commit was made
- the path of the new build document
- the observed behaviour of the unit, end-to-end, canary and timeout tests
