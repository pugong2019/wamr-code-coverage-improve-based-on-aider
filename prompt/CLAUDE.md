# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Key Directories

- **core/**: Core WAMR runtime implementation
- **product-mini/**: `iwasm` executable and platform-specific code
- **wamr-compiler/**: AOT compiler source
- **samples/**: Example applications and usage patterns
- **tests/**: Unit tests, benchmarks, and regression tests
- **doc/**: Comprehensive documentation
- **build-scripts/**: CMake build configuration scripts

## Primary Focus: Unit Testing and Code Coverage

### Current LCOV Coverage Status (Updated 2025-09-08)
- **Line Coverage**: 17,053 / 46,936 (36.3%)
- **Function Coverage**: 1,546 / 3,359 (46.0%)
- **Branch Coverage**: 9,456 / 46,943 (20.1%)
- **Report Location**: `tests/unit/wamr-lcov/index.html`
- **Last Updated**: 2025-09-08 13:18:04

### Unit Testing with GTest Framework and Follow Current Unit Code Convention

#### Build and Run Unit Tests(CRITICAL)
```bash
# Build unit tests with coverage
cd tests/unit/{module}
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug -DWAMR_BUILD_COVERAGE=1
make
./{unit_tests_module_name}

# Generate coverage report
lcov --capture --directory . --output-file coverage.info
genhtml coverage.info --output-directory coverage_report
```

#### Unit Test Structure
- All unit tests use **Google Test (GTest)** framework
- Test files located in `tests/unit/[module_name]/`
- If module directory doesn't exist, create it following the pattern
- Common test utilities in `tests/unit/common/test_helper.h`

#### GTest Code Conventions
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

#### Creating New Unit Test Modules
1. Create directory: `tests/unit/[module_name]/`
2. Add `CMakeLists.txt` following existing patterns
3. Create test files: `test_[feature].cpp`
4. Include in main unit test build via `tests/unit/CMakeLists.txt`

## Core Modules for Unit Testing

### Priority Areas for Coverage Improvement
Based on current coverage (36.3% lines, 46.0% functions, 20.1% branches):

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
When user specifies a target module and target code coverage (e.g., `interpreter`, `aot`, `runtime-common`):

```bash
# 1. Analyze module coverage from current LCOV report
# Navigate to coverage report and examine specific module
firefox coverage_report/wamr-lcov/index.html
# Or examine specific module path: coverage_report/wamr-lcov/wasm-micro-runtime/core/iwasm/[module]/

# 2. Extract uncovered functions and lines for the module
grep -A 5 -B 5 "uncovered" coverage_report/wamr-lcov/wasm-micro-runtime/core/iwasm/[module]/index.html
```

### Phase 2: Test Plan Generation
Create a systematic sub-plan in `tests/unit/[module_name]/coverage_improve_plan.md`:

```markdown
# Test Plan for [Module Name]

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
```bash
# 1. Generate test cases for current step (≤10 cases)
# Create tests in tests/unit/[module_name]/test_[step_name].cpp and related CMakeLists.txt (If need)

# 2. Build with coverage
cd tests/unit/{module_name}
rm -rf build/  # Clean previous build
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug -DWAMR_BUILD_COVERAGE=1
make

# 3. Run tests immediately after generation
./unit_tests --gtest_filter="*[ModuleName]*[StepName]*"

# 4. Generate coverage report for verification
lcov --capture --directory . --output-file step_coverage.info
genhtml step_coverage.info --output-directory step_coverage_report

# 5. Verify coverage improvement and fix issues if any
# Check step_coverage_report/index.html for new coverage

# 6. Update test plan status
# Mark current step as COMPLETED in coverage_improve_plan.md
# Move to next step
```

### Phase 4: Issue Resolution Protocol
If build/test issues occur:

```bash
# 1. Fix compilation errors
# - Check include paths
# - Verify test_helper.h usage
# - Fix syntax errors

# 2. Fix test failures
# - Review WAMR API usage
# - Check test logic and assertions
# - Verify test data and setup

# 3. Re-run build and test cycle
make && ./unit_tests --gtest_filter="*[FailedTest]*"

# 4. Continue only after all tests pass
```

### Phase 5: Progress Tracking
For each module, maintain status in `tests/unit/[module_name]/coverage_improve_plan.md`:

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
```bash
# Start coverage improvement for a module
cd tests/unit/[module_name]/
# Create coverage_improve_plan.md if not exists
# Implement Step 1 test cases
# Build, test, verify coverage
# Update coverage_improve_plan.md status
# Proceed to Step 2

# Quick verification cycle
make && ./unit_tests --gtest_filter="*[ModuleName]*" && \
lcov --capture --directory . --output-file temp.info && \
genhtml temp.info --output-directory temp_report
```