# AGENT.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.  

The goal is to generate additional comprehensive unit tests to increase the **codelines coverage** against the source file.

## Key Directories

- **core/**: Core WAMR runtime implementation
- **product-mini/**: `iwasm` executable and platform-specific code
- **wamr-compiler/**: AOT compiler source
- **samples/**: Example applications and usage patterns
- **tests/**: Unit tests, benchmarks, and regression tests
- **doc/**: Comprehensive documentation
- **build-scripts/**: CMake build configuration scripts

## Ignored Directories

- **language-bindings/**
- **zephyr/**
## Coverage Report Location
**Report Location**: `tests/unit/wamr-lcov/wamr-lcov/index.html`

### Overall Current LCOV Coverage Status
- **Line Coverage**: 
- **Function Coverage**:
- **Branch Coverage**:
- **Last Updated**: 2025-08-24 08:33:09

## Key Conduct Principles (CRITICAL)

### Test Framework and Code Convention 
- Test framework: GTest Framework 
- Code convention: Follow Current Unit Code Convention

#### Build and Run Unit Tests
```bash
# Build unit tests with coverage
cd tests/unit/{module}
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug -DWAMR_BUILD_COVERAGE=1
make
./[EXECUTABLE_MODULE_NAME]

# Generate coverage report
lcov --capture --directory . --output-file coverage.info
genhtml coverage.info --output-directory coverage_report
```

### Unit Test Structure
- All unit tests use **Google Test (GTest)** framework
- Test files located in `tests/unit/[ModuleName]/`
- If module directory doesn't exist, create it following the pattern
- Common test utilities in `tests/unit/common/test_helper.h`

### GTest Code Conventions
```cpp
// Use test_helper.h for WAMR-specific utilities
#include "test_helper.h"

// Test class naming: [ModuleName]Test
class WasmRuntimeTest : public ::testing::Test {
protected:
    WAMRRuntimeRAII<> runtime;  // Auto-initialize WAMR runtime
    
    void SetUp() override {
        // Test setup code
    }
    
    void TearDown() override {
        // Cleanup code
    }
};

// Test naming: TEST_F(TestClass, TestName)
TEST_F(WasmRuntimeTest, LoadValidModule) {
    // Test implementation
    WAMRModule module(test_wasm_buffer, sizeof(test_wasm_buffer));
    EXPECT_NE(module.get(), nullptr);
}

// Use WAMR test utilities:
// - WAMRRuntimeRAII: Auto-manage runtime lifecycle
// - WAMRModule: RAII wrapper for wasm_module_t
// - WAMRInstance: RAII wrapper for wasm_module_inst_t  
// - WAMRExecEnv: RAII wrapper for wasm_exec_env_t
// - DummyExecEnv: Complete test environment setup
```

### Creating New Unit Test Modules
1. Create directory: `tests/unit/[ModuleName]/`
2. Add `CMakeLists.txt` following existing patterns
3. Create test files: `test_[feature].cpp`
4. Include in main unit test build via `tests/unit/CMakeLists.txt`

## Core Modules for Unit Testing

### Priority Areas for Coverage Improvement
1. **Runtime Common** (`core/iwasm/common/`): Core runtime APIs and utilities
2. **Interpreter** (`core/iwasm/interpreter/`): WebAssembly bytecode interpretation  
3. **AOT Runtime** (`core/iwasm/aot/`): Ahead-of-Time compiled module execution
4. **Memory Management**: Linear memory, heap, and stack operations
5. **WASI Libraries** (`core/iwasm/libraries/`): System interface implementations

### Key Testing Targets
- **Memory Operations**: Allocation, bounds checking, address translation
- **Module Loading**: Validation, instantiation, symbol resolution  
- **Function Calls**: Parameter passing, return values, exception handling
- **WASI Functions**: File I/O, networking, threading primitives
- **Error Conditions**: Invalid modules, out-of-memory, stack overflow

### Existing Unit Test Modules
Current test directories in `tests/unit/`:
- `aot/`, `aot-stack-frame/`: AOT runtime testing
- `interpreter/`: Interpreter functionality  
- `runtime-common/`: Common runtime operations
- `memory64/`, `linear-memory-*/`: Memory system testing
- `shared-heap/`, `shared-utils/`: Memory sharing features
- `gc/`: Garbage collection (experimental)
- `compilation/`: AOT compilation pipeline

## Coverage Improvement Workflow (CRITICAL)

### Phase 1: Module Analysis
1. When user specifies a target module and target code coverage (e.g., `interpreter`, `aot`, `runtime-common`):
    - Analyze module coverage from current LCOV report
    - Navigate to coverage report and examine specific module
    ```bash
    firefox coverage_report/wamr-lcov/index.html
    # Or examine specific module path: coverage_report/wamr-lcov/wasm-micro-runtime/core/iwasm/[module]/
    ```
2. Extract uncovered functions and lines for the module
    ```bash
    grep -A 5 -B 5 "uncovered" coverage_report/wamr-lcov/wasm-micro-runtime/core/iwasm/[module]/index.html
    ```

### Phase 2: Test Plan Generation
Create a systematic sub-plan in `tests/unit/[ModuleName]//[ModuleName]_coverage_improve_plan.md`:

```markdown
# Code Coverage Improve Plan for [Module Name]

## Current Coverage Status
- Line Coverage: X/Y (Z%)
- Function Coverage: A/B (C%)
- Branch Coverage: D/E (F%)

## Test Generation Sub-Plans

### Function Segmentation Strategy
For large code file with >50 uncovered lines with 5 functions, segment uncovered code by function signatures:
1. **Analyze Coverage**: Extract uncovered functions from LCOV report
2. **Group Functions**: Group related functions into logical segments (≤5 functions per segment)
3. **Create Steps**: Each step targets one segment with ≤5 test cases maximum
4. **Sequential Execution**: Complete one step fully before proceeding to next

### Step Planning Formula
- **Total Functions**: Count uncovered functions needing test coverage
- **Step Size**: Maximum 5 test cases per step (covering ≤5 functions)
- **Step Count**: `ceil(total_functions / 10)` steps required
- **Example**: 30 uncovered functions = 3 steps (10 functions each)

### Step Template Structure
#### Step N: [Segment Name] Functions (≤10 test cases)
**Target Functions**: `function_name1()`, `function_name2()`, ..., `function_name10()`
- [ ] test_function_name1_basic_operation  
- [ ] test_function_name1_error_handling
- [ ] test_function_name2_success_path
- [ ] test_function_name2_boundary_conditions
- [ ] test_function_name3_null_input_validation
- [ ] test_function_name4_memory_operations
- [ ] test_function_name5_parameter_validation
- [ ] test_function_name6_return_value_checks
- [ ] test_function_name7_edge_case_scenarios
- [ ] test_function_name8_integration_flows
- **Status**: PENDING/IN_PROGRESS/COMPLETED
- **Coverage Target**: Cover all lines in listed functions
- **Completion Criteria**: All 10 test cases pass + coverage report shows improvement

### Multi-Step Execution Protocol
1. **Step Preparation**: Identify target function segment for current step
2. **Test Generation**: Create ≤10 test cases covering target functions only  
3. **Build & Test**: Compile and run current step tests in isolation
4. **Coverage Verification**: Generate step-specific coverage report
5. **Status Update**: Mark step as COMPLETED only when all criteria met
6. **Next Step**: Proceed to next function segment only after current step completion
```

### Phase 3: Step-by-Step Execution

#### Execute One Step at a Time

1. Generate test cases for current step (≤10 cases)
 Create tests in tests/unit/[ModuleName]/test_[step_name].cpp and related CMakeLists.txt (If need)

2. Build with coverage
```bash
cd tests/unit/[ModuleName]
rm -rf build/  # Clean previous build
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug -DWAMR_BUILD_COVERAGE=1
make
```

3. Run tests immediately after building successfully
    ```bash
    ./[EXECUTABLE_MODULE_NAME] --gtest_filter="*[ModuleName]*[StepName]*"
    ```

4. Generate coverage report for verification
    ```bash
    lcov --capture --directory . --output-file step_coverage.info
    genhtml step_coverage.info --output-directory step_coverage_report
    ```

5. Verify coverage improvement and fix issues if any
6. Check step_coverage_report/index.html for new coverage

7. Update test plan status:   
Mark current step as COMPLETED in [ModuleName]_coverage_improve_plan.md
8. Move to next step in the plan

### Phase 4: Issue Resolution Protocol
If build/test issues occur:
1. Fix CMakeLists.txt build errors
    - FIRST: Refer to other unit test module's CMakeLists.txt for patterns
    - Check existing modules: aot/, shared-utils/, interpreter/, runtime-common/,and so on
    - Copy working CMake configuration and adapt for current module
    - Verify WAMR_BUILD_* flags match working examples

2. Fix compilation errors
    - Check include paths
    - Verify test_helper.h usage
    - Fix syntax errors

3. Fix test failures
    - Review WAMR API usage
    - Check test logic and assertions
    - Verify test data and setup

4. Re-run build and test cycle
    ```bash
    make && ./[EXECUTABLE_NAME] --gtest_filter="*[FailedTest]*"
    ```
5. Continue only after all tests pass

### Phase 5: Progress Tracking
For each module, maintain status in `tests/unit/[ModuleName]/[[EXECUTABLE_NAME]]coverage_improve_plan.md`:

```markdown
## Overall Progress
- Total Steps: X
- Completed Steps: Y
- Current Step: Z
- Module Coverage Before: A%
- Module Coverage After: B%
- Target Coverage: C%

## Step Status
- [x] Step 1: Core Functions - COMPLETED (Date: YYYY-MM-DD)
- [x] Step 2: Error Handling - COMPLETED (Date: YYYY-MM-DD) 
- [ ] Step 3: Edge Cases - IN_PROGRESS
- [ ] Step 4: Integration Tests - PENDING
```

### Workflow Commands Summary

Start coverage improvement for a module
```bash
cd tests/unit/[ModuleName]/
```
1. Create [ModuleName]_coverage_improve_plan.md if not exists or not the date is not latest
2. Implement Step 1 test cases code generate
3. Build, fix, test
4. verify coverage
    ```bash
    make && ./unit_tests --gtest_filter="*[ModuleName]*" && \
    lcov --capture --directory . --output-file temp.info && \
    genhtml temp.info --output-directory temp_report
    ```
5. Update [ModuleName]_coverage_improve_plan.md status
6. Proceed to Step 2

##  Final Coverage Collection
Once all phases are complete and all test steps are implemented, build and run the entire module test suite to collect overall code coverage:

```bash
# From the module test directory
cd wasm-micro-runtime/tests/unit
# Configure build with coverage enabled
cmake -S . -B build -DCOLLECT_CODE_COVERAGE=1
# Build all tests
cmake --build build
# Run all tests
ctest --test-dir build
# Collect and generate the final coverage report
../wamr-test-suites/spec-test-script/collect_coverage.sh unit.lcov ./build/
```