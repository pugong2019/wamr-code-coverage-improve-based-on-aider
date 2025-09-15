# CMake Error Fix Collection

This document collects common CMake build errors and their solutions for the WAMR project.

## 1. GoogleTest Duplicate Fetching Error

### Error Description
```
fatal error: gmock/gmock.h: No such file or directory
   39 | #include "gmock/gmock.h"
      |          ^~~~~~~~~~~~~~~
compilation terminated.
```

### Root Cause
When building the entire unit test suite, GoogleTest is fetched multiple times:
1. First in the main `tests/unit/CMakeLists.txt` 
2. Again in individual module's `unit_common.cmake`

This creates conflicts where GMock headers cannot be found due to duplicate/conflicting GoogleTest configurations.

### Solution Steps

#### Step 1: Fix Target Linking
Remove extra `gtest` library and redundant GoogleTest setup:

**Before:**
```cmake
add_executable (aot_test ${unit_test_sources})
enable_testing()
include(GoogleTest)
target_link_libraries (aot_test ${LLVM_AVAILABLE_LIBS} gtest_main gtest)
```

**After:**
```cmake
add_executable (aot_test ${unit_test_sources})

target_link_libraries (aot_test ${LLVM_AVAILABLE_LIBS} gtest_main)
```

#### Step 2: Prevent Duplicate GoogleTest Fetching
Add GoogleTest inclusion guard before including `unit_common.cmake`:

**Add to module's CMakeLists.txt:**
```cmake
# Set GOOGLETEST_INCLUDED to prevent duplicate fetching when building as part of whole unit test suite
if(NOT DEFINED GOOGLETEST_INCLUDED)
    set(GOOGLETEST_INCLUDED 0)
endif()

include (../unit_common.cmake)
```

### Affected Files
- `/tests/unit/[module]/CMakeLists.txt` (any module with this issue)
- Main unit test builds that include multiple modules

### Prevention
- Always check if GoogleTest is already configured before fetching
- Use `gtest_main` only (includes core gtest functionality)
- Follow the pattern used by working modules like `shared-heap`, `runtime-common`

---

## Template for Future Fixes

### Error Description
[Brief description of the error message]

### Root Cause
[Explanation of why the error occurs]

### Solution Steps
[Step-by-step fix instructions]

### Affected Files
[List of files that need changes]

### Prevention
[How to avoid this error in the future]

---