# Test Plan Generation Guide

This guide helps create systematic test plans for any WAMR module to achieve specific code coverage targets following CLAUDE.md methodology.

## Prerequisites

Before creating a test plan, ensure you have:
1. **LCOV coverage report** available at `tests/unit/wamr-lcov/index.html`
2. **Target module** identified (e.g., `interpreter`, `aot`, `runtime-common`, `simd`)
3. **Target coverage percentage** defined (e.g., 10%, 25%, 50%)
4. **Module source location** known (e.g., `core/iwasm/interpreter/`)

## Step-by-Step Plan Generation Process

### Phase 1: Module Coverage Analysis

#### 1.1 Extract Current Coverage Data
```bash
# Navigate to module-specific coverage report
firefox tests/unit/wamr-lcov/wasm-micro-runtime/core/iwasm/[MODULE_NAME]/index.html

# Extract coverage statistics
grep -E "(Lines|Functions|Branches)" tests/unit/wamr-lcov/wasm-micro-runtime/core/iwasm/[MODULE_NAME]/index.html
```

#### 1.2 Analyze Uncovered Functions
```bash
# Extract uncovered function names from .func.html files
find tests/unit/wamr-lcov/wasm-micro-runtime/core/iwasm/[MODULE_NAME]/ -name "*.func.html" -exec grep -l "coverFnLo.*>0<" {} \;

# Count total uncovered functions
grep -r "coverFnLo.*>0<" tests/unit/wamr-lcov/wasm-micro-runtime/core/iwasm/[MODULE_NAME]/ | wc -l
```

#### 1.3 Document Module Structure
Identify key source files and their function counts:
- List all `.c/.cc/.cpp` and `.h` files in the module
- Count uncovered functions per file
- Identify high-priority/foundational functions

### Phase 2: Calculate Step Planning Metrics

#### 2.1 Coverage Target Calculation
```
Target Lines = (Target Coverage % × Total Lines) / 100
Target Functions = (Target Coverage % × Total Functions) / 100
```

#### 2.2 Step Segmentation Formula
```
Step Count = ceil(Target Functions / 10)
Functions Per Step = min(10, remaining_functions)
```

#### 2.3 Priority Function Selection
Prioritize functions based on:
1. **Foundational functions** (used by many other functions)
2. **High line-count functions** (maximum coverage impact)
3. **Error handling functions** (critical for robustness)
4. **Public API functions** (external interface coverage)

### Phase 3: Generate {module_name}_coverage_improve_plan.md Template

#### 3.1 Create Test Plan File
```bash
# Create module test directory if needed
mkdir -p tests/unit/[MODULE_NAME]/

# Create coverage_improve_plan.md from template
touch tests/unit/[MODULE_NAME]/{_module_name}_coverage_improve_plan.md
```

#### 3.2 Test Plan Template Structure

```markdown
# Code Coverage Improve Plan for [Module Name]

## Current Coverage Status
- Line Coverage: [CURRENT_COVERED]/[TOTAL_LINES] ([CURRENT_PERCENT]%)
- Function Coverage: [CURRENT_COVERED]/[TOTAL_FUNCTIONS] ([CURRENT_PERCENT]%)
- Branch Coverage: [CURRENT_COVERED]/[TOTAL_BRANCHES] ([CURRENT_PERCENT]%)

## Test Generation Sub-Plans

### Function Segmentation Strategy
For [MODULE_NAME] module with [TOTAL_UNCOVERED] uncovered functions, segment uncovered code by function signatures:
1. **Analyze Coverage**: Extracted [TOTAL_UNCOVERED] uncovered functions from LCOV report
2. **Group Functions**: Grouped into logical segments (≤10 functions per segment)
3. **Create Steps**: [STEP_COUNT] steps targeting [TARGET_FUNCTIONS] functions total for [TARGET_PERCENT]% coverage
4. **Sequential Execution**: Complete one step fully before proceeding to next

### Step Planning Formula
- **Total Functions**: [TOTAL_UNCOVERED] uncovered functions needing test coverage
- **Step Size**: Maximum 10 test cases per step (covering ≤10 functions)
- **Step Count**: [STEP_COUNT] steps required for [TARGET_PERCENT]% target ([TARGET_FUNCTIONS] functions)
- **Coverage Target**: [TARGET_PERCENT]% (≈[TARGET_LINES] lines out of [TOTAL_LINES] total)

### Step Template Structure

#### Step 1: [FUNCTIONAL_GROUP_NAME] Functions (≤10 test cases)
**Target Functions**: `function1()`, `function2()`, `function3()`, ..., `function10()`  
- [ ] test_function1_basic_operation  
- [ ] test_function1_error_handling
- [ ] test_function2_success_path
- [ ] test_function2_boundary_conditions
- [ ] test_function3_null_input_validation
- [ ] test_function4_memory_operations
- [ ] test_function5_parameter_validation
- [ ] test_function6_return_value_checks
- [ ] test_function7_edge_case_scenarios
- [ ] test_function8_integration_flows
- **Status**: PENDING/IN_PROGRESS/COMPLETED
- **Coverage Target**: Cover all lines in listed functions (~[ESTIMATED_LINES] lines)
- **Completion Criteria**: All 10 test cases pass + coverage report shows [STEP_COVERAGE]% improvement

[Repeat for each step...]

### Multi-Step Execution Protocol
1. **Step Preparation**: Identify target function segment for current step
2. **Test Generation**: Create ≤10 test cases covering target functions only  
3. **Build & Test**: Compile and run current step tests in isolation
4. **Coverage Verification**: Generate step-specific coverage report
5. **Status Update**: Mark step as COMPLETED only when all criteria met
6. **Next Step**: Proceed to next function segment only after current step completion

## Overall Progress
- Total Steps: [STEP_COUNT]
- Completed Steps: 0
- Current Step: 1 (PENDING)
- Module Coverage Before: [CURRENT_PERCENT]%
- Module Coverage After: [CURRENT_PERCENT]%
- Target Coverage: [TARGET_PERCENT]%

## Step Status
- [ ] Step 1: [GROUP1_NAME] Functions - PENDING
- [ ] Step 2: [GROUP2_NAME] Functions - PENDING
[... for each step]

## Implementation Notes
- **Test Framework**: Google Test (GTest) following current unit code convention
- **Test Location**: `tests/unit/[MODULE_NAME]/`
- **Coverage Tool**: LCOV
- **Build Commands**:
```bash
# Build unit tests with coverage
cd tests/unit/[MODULE_NAME]
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug -DWAMR_BUILD_COVERAGE=1
make
./[EXECUTABLE_NAME] --gtest_filter="*[MODULE_NAME]*"

# Generate coverage report
lcov --capture --directory . --output-file coverage.info
genhtml coverage.info --output-directory coverage_report
```
- **Required Libraries**: [MODULE_SPECIFIC_DEPENDENCIES]
- **Test Pattern**: Follow existing `tests/unit/[SIMILAR_MODULE]/` structure
```
```

### Phase 4: Implementation Guidelines

#### 4.1 Test File Structure
```cpp
/*
 * Copyright (C) 2019 Intel Corporation. All rights reserved.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 */

#include "test_helper.h"
#include "gtest/gtest.h"

#include "[MODULE_HEADER].h"
#include "[DEPENDENCY_HEADERS].h"

class [ModuleName]Test : public ::testing::Test {
protected:
    WAMRRuntimeRAII<> runtime;  // Auto-initialize WAMR runtime
    // Add module-specific setup
    
    void SetUp() override {
        // Module-specific setup code
    }
    
    void TearDown() override {
        // Cleanup code
    }
};

// Step 1 test cases
TEST_F([ModuleName]Test, Function1BasicOperation) {
    // Test implementation
}
```

#### 4.2 Build Integration
Create or update `CMakeLists.txt` in module test directory:
```cmake
# Include existing compilation module pattern
include(../unit_common.cmake)

# Add module-specific includes
include_directories(${MODULE_INCLUDE_DIRS})

# Add module-specific sources
set(MODULE_TEST_SOURCES
    test_[module]_step1.cpp
    test_[module]_step2.cpp
    # ... other test files
)

# Create executable
enable_testing()
include(GoogleTest)
add_executable([module]_test ${MODULE_TEST_SOURCES} ${COMMON_SOURCES})
target_link_libraries([module]_test ${REQUIRED_LIBS} gtest_main)

gtest_discover_tests([module]_test)
```
#### 4.3 Final Coverage Collection

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

This process ensures all test cases are executed and the latest coverage data is collected for comprehensive analysis.

## Plan Generation Checklist

- [ ] Module coverage data extracted and analyzed
- [ ] Total uncovered functions counted
- [ ] Coverage target and step count calculated  
- [ ] Functions grouped into logical segments (≤10 per step)
- [ ] coverage_improve_plan.md created with all required sections
- [ ] Step templates filled with actual function names
- [ ] Build commands updated for module-specific paths
- [ ] Implementation notes customized for module requirements
- [ ] Overall progress tracking initialized
- [ ] Step status checkboxes created for all steps


## Quality Assurance

Before finalizing a test plan:
1. **Verify function names** exist in actual source code
2. **Check coverage calculations** are mathematically correct
3. **Ensure step distribution** is balanced (functions per step)
4. **Validate build commands** for module-specific requirements
5. **Review grouping logic** for functional coherence

This guide ensures consistent, systematic test plan generation for any WAMR module following the established CLAUDE.md methodology.