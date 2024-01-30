# SurrealDB Reproduce

This utility is an aid to locally reproduce crashes found by [OSS-Fuzz](https://oss-fuzz.com/) in SurrealDB. It aims to reduce the burden of [reproducing crashes in a local environment](https://google.github.io/oss-fuzz/advanced-topics/reproducing/), provide additional information (e.g. plain-text queries for binary test cases), allow developers to quickly iterate on potential fixes and identify if the crash can be triggered remotely.

## Context

### Fuzzing

Currently, OSS-Fuzz is fuzzing the SurrealDB legacy parser and executor. The parser is fuzzed by the fuzzing target `fuzz_sql_parser` by generating variations of SurrealQL query strings continously mutated to maximize code coverage. The executor is fuzzed similarly by `fuzz_executor`. The `fuzz_structured_executor` target also fuzzes the executor but directly generates the AST of pre-parsed queries (i.e. `sql::Query`) instead of query strings; this allows for better performance generating valid test cases which results in increased effective coverage.

### Reporting

Issues found by OSS-Fuzz in SurrealDB are initially privately reported and only accessible to the members listed on [the OSS-Fuzz SurrealDB project](https://github.com/google/oss-fuzz/blob/master/projects/surrealdb/project.yaml#L5). After a 90 day reporting deadline elapses, issues are made public an visible by anyone. Issues both public and private (for those with permissions) can be found in [the Chromium issue tracker](https://oss-fuzz.com/testcases?open=yes&project=surrealdb). The specific test cases that triggered the crashes can be found in [the OSS-Fuzz interface](https://oss-fuzz.com/testcases?open=yes&project=surrealdb).

## Reproducing

Here are the suggested steps to reproduce crashes locally using `surrealdb-reproduce`.

1. Download the relevant test case [from OSS-Fuzz](https://oss-fuzz.com/testcases?open=yes&project=surrealdb).
2. Identify if the test case contains binary data (i.e. it was produced by the `fuzz_structured_executor` target) or a plain-text query.
3. Run this tool using `cargo run` and provide the path to the test case. This will run the test case against the locally checked out version of SurrealDB.
   - If the test case contains a plain-text query string, provide the `-s` flag (e.g. `cargo run test_cases/example_1 -- -s`) to perform parsing.
4. The crash should trigger and some relevant information will be printed that should help you identify the input query and the location of the crash.
   - If the output does not contain sufficient information, provide the `-b` flag (e.g. `cargo run test_cases/example_1 -- -b`) to get a full backtrace.
   - If you want to identify if the crash can be triggered remotely, provide the `-r` flag (e.g. `cargo run test_cases/example_1 -- -r`) to spawn a server.
5. Amend the code to resolve the bug (or add any log traces necesary to debug it) and run the tool again to attempt to reproduce the crash with the new changes.

## Alternatives

If the crash does not reproduce with the method described above, it may be because of differences between your local enviromnent and OSS-Fuzz. These differences can be minimized by following the [process to reproduce described by OSS-Fuzz](https://google.github.io/oss-fuzz/advanced-topics/reproducing/) instead of using this tool.