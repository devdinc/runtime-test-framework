# Runtime Test Framework

A runtime testing framework for Skript, built around Skript's test-tracking system. It lets scripts declare tests that can be registered, discovered, and executed while the server is running, rather than requiring Skript's development test runner.

The framework uses its own custom-event pipeline for test execution and error tracking. When Skript's internal `TestTracker` is available, it also forwards test-start and test-failure notifications to it through reflection. This integration is optional and does not replace the framework's own runtime test system.

## Requirements

- Skript 2.15+
- skript-reflect 2.6+

## Installation

```sh
package install https://github.com/devdinc/runtime-test-framework
```

Load the framework before the scripts containing its test declarations. When installed through the package workflow, put test files in the package's `-tests/` directory.

## Declaring tests

Use `devdinc test` to define a test:

```skript
devdinc test "addition works":
    assert 1 + 1 is 2
```

Tests are registered during script parsing and associated with the script that declares them. They are automatically executed after scripts load. The optional `when` condition is checked immediately before the test runs:

```skript
devdinc test "only when enabled" when {testing-enabled} is true:
    assert 2 + 2 is 4
```

## Assertions and failures

Assertions evaluate conditions inside the test event. A failed assertion records an error and halts the current test by default.

```skript
devdinc test "assertion examples":
    assert {_value} is set
    assert {_value} is greater than 0 with "value should be positive"
    without halting assert {_other} is set
    assert {_flag} is false
    fail test with "expected this test to fail"
```

- `without halting` records the failure and continues the test.
- `fail test` explicitly records a failure.
- Assertions and `fail test` accept an optional error message.
- Use `with no error message` to suppress the failure message.

## Discovering and running tests

Tests can be listed by name or by their source script:

```skript
set {_tests::*} to all tests
set {_named::*} to all tests with test name "addition works"
set {_script-tests::*} to all tests in script "example.sk"
```

Run the selected test objects manually:

```skript
run {_tests::*}
```

Use `autorun {_tests::*}` to execute them with the autorun flag enabled. The framework also runs registered tests automatically after scripts load.

Each test is identified by its name and originating script, so different scripts may use the same test name.

## Test event values

Tests and lifecycle hooks expose:

| Event value | Type | Meaning |
|---|---|---|
| `event-string` | string | Test name |
| `event-script` | script | Script that defines the test |
| `event-boolean` | boolean | Whether this is an autorun |
| `event-test` | object | Pair containing the test name and script |

These values are guaranteed when tests are declared with `devdinc test`. They may be unavailable or incomplete when running under Skript's native test runner.

Check execution mode with:

```skript
if event-test is autorun:
    stop auto test execution here
```

`stop auto test execution here` only stops the current test during autorun; it has no effect during manual execution.

## Lifecycle hooks

The framework provides hooks for setup and cleanup:

- `before each test`
- `after each test`
- `before all tests`
- `after all tests`

The per-test hooks run around each test. The per-script hooks run once for each script whose tests are being executed. In these hooks, `event-script` identifies the script whose tests are running.

```skript
before all tests:
    broadcast "Starting tests in %event-script%"

before each test:
    delete {-temporary-test-state::*}

after each test:
    delete {-temporary-test-state::*}
```

## Parser and syntax testing

The framework can load Skript nodes during a test and capture parser logs. This makes it possible to test syntax and expected parser errors as well as runtime behavior.

```skript
devdinc test "invalid syntax is reported":
    parse:
        this is not valid skript
    assert last parse logs is set
```

Use `latest parse logs` to inspect the captured parser output. The parser state is backed up and restored around this operation.

## Skript native test compatibility

The framework can coexist with Skript's native `quickTest` and `test-action` systems, but the native test event takes precedence in that environment. Core assertion syntax may still work, while framework-specific event values, hooks, autorun controls, and parser helpers may be unavailable or incomplete.

Use `devdinc test` when you need the framework's own custom-event pipeline and extended runtime behavior.

## TestTracker integration

When Skript's internal `ch.njol.skript.test.runner.TestTracker` class is present, the framework forwards test-start and test-failure notifications to it using reflection. This is best-effort, depends on internal class and method signatures, and may vary between Skript versions.

The framework maintains its own registration, execution, and error tracking regardless of whether this optional integration is available.

## Source and tests

- Implementation: `src/runtime-test-framework.sk`
- Framework tests: `-tests/`
