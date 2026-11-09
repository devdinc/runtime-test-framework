# Runtime Test Framework

A lightweight test framework for Skript scripts. Tests are declared in Skript files and can be run automatically on load or manually from Skript.

## Requirements

- Skript 2.15+
- skript-reflect 2.6+

## Installation

```
package install https://github.com/devdinc/runtime-test-framework
```

The framework registers the test syntax and automatically runs registered tests after scripts load.

## Writing tests

Declare a test with `devdinc test`:

```skript
devdinc test "addition works":
    assert 1 + 1 is 2
```

Assertions are conditions. A test passes when the condition evaluates to true; a failed assertion records an error and halts the test by default.

### Assertions

```skript
devdinc test "assertion examples":
    assert {_value} is set
    assert {_value} is greater than 0 with "value should be positive"
    without halting assert {_other} is set
    assert {_flag} is false
    fail test with "expected this test to fail"
```

Use `without halting` to record a failure and continue executing the test. `fail test` explicitly marks the current test as failed. Both assertions and `fail test` accept an optional error message; `with no error message` suppresses the message.

### Test hooks

The framework exposes custom events for setup and teardown:

- `before each test`
- `after each test`
- `before all tests`
- `after all tests`

These events expose the test name and script through their event values. Use them to prepare shared state or clean it up around test execution.

### Listing and running tests

```skript
set {_tests::*} to all tests
set {_named::*} to all tests with test name "addition works"
run {_tests::*}
autorun {_tests::*}
```

Use `run` to execute tests manually and `autorun` to mark execution as an automatic run. Tests can inspect whether the current run is automatic using `event-test is autorun`.

Other helpers include `test errors`, `last test result`, `parse`, and `latest parse logs`.

## Test files

Place tests in your package's `-tests/` directory when using this repository's package layout.

## Source

- Implementation: `src/runtime-test-framework.sk`
- Framework tests: `-tests/`
