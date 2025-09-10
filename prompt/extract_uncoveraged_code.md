# Uncovered Code Analysis Prompt Template

## Task
Analyze the LCOV coverage report for a specific source file and extract all uncovered code lines with their containing function context.

## Input Parameters
- **Source File**: `{filename}` (e.g., `core/iwasm/aot/aot_loader.c`)
- **LCOV Report Path**: `{lcov_report_path}` (e.g., `tests/unit/wamr-lcov/index.html`)

## Instructions
1. **Navigate to Coverage Report**: Open the LCOV HTML report and locate the specific file `{filename}`
2. **Extract Uncovered Lines**: Identify all lines marked as uncovered (typically highlighted in red or marked with line numbers)
3. **Function Context Analysis**: For each uncovered line, determine its containing function by:
   - Finding the function signature that contains the uncovered line
   - Including the complete function signature with parameters
   - Preserving the original code structure and comments
4. **Output Format**: Create a markdown file named `{filename}_uncoveraged_code.md` with the following structure:

## Output Template
```markdown
# Uncovered Code Analysis: {filename}

## Coverage Summary
- **File**: {filename}
- **Total Lines**: {total_lines}
- **Covered Lines**: {covered_lines}
- **Uncovered Lines**: {uncovered_lines}
- **Coverage Percentage**: {coverage_percentage}%

## Uncovered Code Segments

### Function: {function_name_1}
```c
return_type function_name_1(param1_type param1, param2_type param2, ...) {
    // ...existing covered code...
    uncovered_code_line_1;  // Line {line_number}
    uncovered_code_line_2;  // Line {line_number}
    // ...existing covered code...
}
```

### Function: {function_name_2}
```c
return_type function_name_2(param1_type param1, ...) {
    // ...existing covered code...
    uncovered_code_line_3;  // Line {line_number}
    // ...existing covered code...
}
```

## Analysis Notes
- Total uncovered functions: {count}
- Most critical uncovered areas: {description}
- Suggested test priorities: {recommendations}


## Usage Example
**Input**: 
- Filename: `core/iwasm/aot/aot_loader.c`
- LCOV Path: `tests/unit/wamr-lcov/index.html`

**Expected Output**: `core/iwasm/aot/aot_loader_uncoveraged_code.md`

## Processing Steps
1. Parse LCOV HTML report for the target file
2. Extract line-by-line coverage data
3. Identify uncovered line ranges
4. Map uncovered lines to their containing functions
5. Format output with complete function context
6. Save as `{filename}_uncoveraged_code.md`