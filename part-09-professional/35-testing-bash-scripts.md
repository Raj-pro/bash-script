# Chapter 35: Testing Bash Scripts

## Introduction

Testing ensures script reliability and catches bugs early. This chapter covers unit testing, integration testing, and continuous testing for Bash scripts.

---

## Manual Testing

### Basic Test Script

```bash
#!/bin/bash
# test-manual.sh

source ./functions.sh

# Test function
test_add() {
    result=$(add 2 3)
    if [ "$result" -eq 5 ]; then
        echo "✓ add function works"
    else
        echo "✗ add function failed: expected 5, got $result"
        return 1
    fi
}

test_add
```

---

## BATS Framework

### Installing BATS

```bash
# Clone repository
git clone https://github.com/bats-core/bats-core.git
cd bats-core
./install.sh /usr/local

# Verify installation
bats --version
```

### Basic BATS Test

```bash
#!/usr/bin/env bats
# test_functions.bats

# Load function library
load functions

@test "addition works" {
    result=$(add 2 3)
    [ "$result" -eq 5 ]
}

@test "subtraction works" {
    result=$(subtract 5 2)
    [ "$result" -eq 3 ]
}

@test "email validation accepts valid email" {
    run validate_email "user@example.com"
    [ "$status" -eq 0 ]
}

@test "email validation rejects invalid email" {
    run validate_email "invalid-email"
    [ "$status" -eq 1 ]
}
```

### Running BATS Tests

```bash
# Run single test file
bats test_functions.bats

# Run all tests in directory
bats tests/

# Run with tap output
bats --tap tests/

# Run with timing
bats --timing tests/
```

---

## Advanced BATS Testing

### Setup and Teardown

```bash
#!/usr/bin/env bats

setup() {
    # Run before each test
    export TEST_DIR=$(mktemp -d)
    export TEST_FILE="$TEST_DIR/test.txt"
}

teardown() {
    # Run after each test
    rm -rf "$TEST_DIR"
}

@test "create file" {
    touch "$TEST_FILE"
    [ -f "$TEST_FILE" ]
}

@test "write to file" {
    echo "test" > "$TEST_FILE"
    [ "$(cat $TEST_FILE)" = "test" ]
}
```

### Testing Output

```bash
#!/usr/bin/env bats

@test "script outputs correct message" {
    run ./script.sh
    
    [ "$status" -eq 0 ]
    [ "$output" = "Expected message" ]
}

@test "script handles errors" {
    run ./script.sh --invalid
    
    [ "$status" -ne 0 ]
    [[ "$output" =~ "ERROR" ]]
}

@test "multi-line output" {
    run ./script.sh
    
    [ "${lines[0]}" = "First line" ]
    [ "${lines[1]}" = "Second line" ]
}
```

---

## Mocking and Stubbing

### Mock External Commands

```bash
#!/usr/bin/env bats

setup() {
    # Create mock directory
    export MOCK_DIR=$(mktemp -d)
    export PATH="$MOCK_DIR:$PATH"
    
    # Create mock command
    cat > "$MOCK_DIR/curl" << 'EOF'
#!/bin/bash
echo '{"status":"ok"}'
EOF
    chmod +x "$MOCK_DIR/curl"
}

teardown() {
    rm -rf "$MOCK_DIR"
}

@test "script uses curl correctly" {
    run ./api_script.sh
    [ "$status" -eq 0 ]
    [[ "$output" =~ "ok" ]]
}
```

---

## Integration Testing

### Multi-Script Tests

```bash
#!/usr/bin/env bats

load test_helper

@test "full deployment workflow" {
    # Setup
    run ./setup.sh test
    [ "$status" -eq 0 ]
    
    # Build
    run ./build.sh
    [ "$status" -eq 0 ]
    [ -f "dist/app.tar.gz" ]
    
    # Deploy
    run ./deploy.sh test
    [ "$status" -eq 0 ]
    
    # Verify
    run curl -s http://localhost:8080/health
    [ "$status" -eq 0 ]
    [[ "$output" =~ "healthy" ]]
    
    # Cleanup
    run ./cleanup.sh test
    [ "$status" -eq 0 ]
}
```

---

## Test Coverage

### Coverage Script

```bash
#!/bin/bash
# coverage.sh

SCRIPT_FILE=$1
TEST_FILE=$2
COVERAGE_DIR="coverage"

mkdir -p "$COVERAGE_DIR"

# Enable coverage
export SHUNIT_COLOR='none'
PS4='+ ${BASH_SOURCE}:${LINENO}:'
set -x

# Run tests
bash -x "$SCRIPT_FILE" 2> "$COVERAGE_DIR/trace.txt"

# Analyze coverage
echo "=== Coverage Report ==="
total_lines=$(grep -c "^" "$SCRIPT_FILE")
executed_lines=$(grep -oE "${SCRIPT_FILE}:[0-9]+" "$COVERAGE_DIR/trace.txt" | \
                 cut -d: -f2 | sort -u | wc -l)

coverage=$(bc <<< "scale=2; $executed_lines * 100 / $total_lines")
echo "Lines executed: $executed_lines / $total_lines"
echo "Coverage: ${coverage}%"
```

---

## Continuous Testing

### Git Pre-Commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

echo "Running tests before commit..."

# Run shellcheck
if command -v shellcheck > /dev/null; then
    shellcheck *.sh
    if [ $? -ne 0 ]; then
        echo "ShellCheck failed!"
        exit 1
    fi
fi

# Run BATS tests
if command -v bats > /dev/null; then
    bats tests/
    if [ $? -ne 0 ]; then
        echo "Tests failed!"
        exit 1
    fi
fi

echo "All checks passed!"
```

### CI Integration (GitHub Actions)

```yaml
# .github/workflows/test.yml
name: Test Scripts

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Install BATS
      run: |
        git clone https://github.com/bats-core/bats-core.git
        cd bats-core
        sudo ./install.sh /usr/local
    
    - name: Install ShellCheck
      run: sudo apt-get install -y shellcheck
    
    - name: Run ShellCheck
      run: shellcheck *.sh
    
    - name: Run Tests
      run: bats tests/
```

---

## Test-Driven Development

### TDD Workflow

```bash
#!/usr/bin/env bats
# test_calculator.bats

# Write test first (will fail)
@test "multiply function exists" {
    run type multiply
    [ "$status" -eq 0 ]
}

@test "multiply returns correct result" {
    run multiply 3 4
    [ "$output" -eq 12 ]
}

# Now implement the function
# functions.sh:
# multiply() {
#     echo $(( $1 * $2 ))
# }
```

---

## Best Practices

1. **Test edge cases** (empty input, special characters)
2. **Test error conditions** (invalid input, missing files)
3. **Use descriptive test names**
4. **Keep tests independent**
5. **Clean up after tests** (use teardown)
6. **Mock external dependencies**
7. **Run tests automatically** (CI/CD)
8. **Measure coverage**
9. **Test in isolated environments**
10. **Document test requirements**

---

## Complete Test Example

```bash
#!/usr/bin/env bats
# comprehensive_test.bats

load test_helper

setup() {
    export TEST_DIR=$(mktemp -d)
    export CONFIG_FILE="$TEST_DIR/config.ini"
}

teardown() {
    rm -rf "$TEST_DIR"
}

@test "config parser handles valid config" {
    cat > "$CONFIG_FILE" << EOF
[database]
host=localhost
port=5432
EOF
    
    run ./parse_config.sh "$CONFIG_FILE"
    [ "$status" -eq 0 ]
    [[ "$output" =~ "localhost" ]]
}

@test "config parser rejects invalid config" {
    echo "invalid config" > "$CONFIG_FILE"
    
    run ./parse_config.sh "$CONFIG_FILE"
    [ "$status" -ne 0 ]
    [[ "$output" =~ "ERROR" ]]
}

@test "config parser handles missing file" {
    run ./parse_config.sh "/nonexistent/file"
    [ "$status" -ne 0 ]
}

@test "config parser validates required fields" {
    cat > "$CONFIG_FILE" << EOF
[database]
host=localhost
# port is missing
EOF
    
    run ./parse_config.sh "$CONFIG_FILE"
    [ "$status" -ne 0 ]
    [[ "$output" =~ "missing.*port" ]]
}
```

---

## Summary

- Use BATS framework for unit testing
- Test both success and failure cases
- Mock external dependencies
- Implement continuous testing
- Measure test coverage
- Follow TDD principles
- Automate testing in CI/CD

---

**Next Chapter:** [36 - Distributing and Versioning Scripts](36-distributing-versioning.md)
