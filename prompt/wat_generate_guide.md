# WAT File Usage Guidelines for WAMR Unit Tests

## Overview
WebAssembly Text Format (WAT) files are essential for WAMR unit testing when precise control over WebAssembly module structure and behavior is required. This document provides comprehensive rules for LLMs to generate effective WAT files for unit tests.

## Core WAT Generation Rules

### 1. Module Structure Template
**Always follow this structure for consistency:**
```wat
(module
  ;; 1. Memory declarations (with size comments)
  ;; 2. Data initialization (if needed)
  ;; 3. Function definitions with clear exports
  ;; 4. Comments explaining test purpose
)
```

### 2. Memory Declaration Patterns
```wat
;; Standard 32-bit memory
(memory 1)                              ;; 64KB (1 page)

;; Memory64 with size documentation
;; Memory definition: 4 GB = 65536
;;                    8 GB = 131072
;;                    16 GB = 262144
(memory (;0;) i64 131072 131072)        ;; 8GB memory for memory64 tests

;; Shared memory for atomic operations
(memory (;0;) i64 200 200 shared)       ;; Shared memory required for atomics

;; Small memory for boundary testing
(memory (;0;) i64 1 1)                  ;; 64KB for out-of-bounds tests
```

### 3. Function Export Naming Convention
**Use descriptive, test-specific names following these patterns:**
- `test_[feature]`: General feature testing
- `[type]_[operation]_[variant]`: Specific operations (e.g., `i64_atomic_store`, `i32_load_offset_4GB`)
- `trigger_[condition]`: Error condition testing (e.g., `trigger_out_of_bounds`)
- `[action]_[target]`: Action-based naming (e.g., `touch_every_page`, `memory_fill_test`)

### 4. Parameter and Local Variable Naming
```wat
;; Use descriptive parameter names
(func (export "test_function") (param $addr i64) (param $value i64) (param $old i64) (param $new i64)
  ;; Use descriptive local variables
  (local $i i64)
  (local $result i32)
)
```

## When to Generate WAT Files

### Required Scenarios (MUST use WAT)
1. **Memory64 Operations**
   - i64 addressing beyond 4GB
   - Large offset operations (>4GB)
   - Memory boundary testing at 8GB+ limits

2. **Atomic Operations**
   - Shared memory with atomic instructions
   - Multi-threaded memory operations
   - Compare-and-swap operations

3. **Edge Case Testing**
   - Memory boundary conditions
   - Integer overflow/underflow scenarios
   - Stack overflow conditions
   - Invalid instruction sequences

4. **WebAssembly Feature Testing**
   - SIMD instructions
   - Reference types
   - Bulk memory operations
   - Exception handling

5. **Error Condition Testing**
   - Malformed module structures
   - Runtime exception triggers
   - Type system violations
   - Resource exhaustion

### Optional Scenarios (CAN use C/C++)
- Simple arithmetic operations
- Standard library functions
- Basic control flow
- Regular application logic

## WAT File Generation Templates

### Memory64 Testing Template
```wat
(module
  ;; Memory definition with size comments
  ;; 8GB = 131072 pages for testing large address space
  (memory (;0;) i64 131072 131072)
  
  ;; Optional: Initialize memory with test pattern
  (data (i64.const 0) "\01\02\03\04\05\06\07\08\09\0A\0B\0C\0D\0E\0F\10")
  
  ;; Test function with 4GB+ offset
  (func (export "i64_load_offset_4GB") (param $addr i64) (result i64)
    ;; Load with 4GB offset to test large addressing
    (i64.load offset=0x100000000 (local.get $addr))
  )
  
  ;; Test function for store operations
  (func (export "i64_store_offset_4GB") (param $addr i64) (param $value i64)
    ;; Store with 4GB offset
    (i64.store offset=0x100000000 (local.get $addr) (local.get $value))
  )
  
  ;; Memory boundary testing function
  (func (export "touch_every_page") (result i64 i64 i32 i32)
    (local $i i64)
    i64.const 0x0000000000000ff8
    local.set $i 
    loop $loop
      ;; Touch memory at each page boundary
      local.get $i
      local.get $i
      i64.store
      local.get $i 
      i64.const 4096
      i64.add 
      local.set $i 
      local.get $i
      ;; Stop before 8GB boundary
      i64.const 0x0000000200000000 
      i64.const 8
      i64.sub
      i64.lt_u 
      br_if $loop
    end
    ;; Return test values for verification
    i64.const 0x000000000000fff8
    i64.load
    i64.const 0x000000010000fff8
    i64.load
    i64.const 0x000000010001fff8
    i32.load
    i64.const 0x000000010001fffc
    i32.load
    return 
  )
)
```

### Atomic Operations Template
```wat
(module
  ;; Shared memory required for atomic operations
  (memory (;0;) i64 200 200 shared)
  
  ;; Initialize memory with test pattern
  (data (i64.const 0) "\01\02\03\04\05\06\07\08\09\0A\0B\0C\0D\0E\0F\10")
  
  ;; i32 atomic operations
  (func (export "i32_atomic_store") (param $addr i64) (param $value i32)
    (i32.atomic.store (local.get $addr) (local.get $value))
  )
  
  (func (export "i32_atomic_load") (param $addr i64) (result i32)
    (i32.atomic.load (local.get $addr))
  )
  
  ;; i64 atomic operations
  (func (export "i64_atomic_store") (param $addr i64) (param $value i64)
    (i64.atomic.store (local.get $addr) (local.get $value))
  )
  
  (func (export "i64_atomic_load") (param $addr i64) (result i64)
    (i64.atomic.load (local.get $addr))
  )
  
  ;; Atomic RMW operations
  (func (export "i64_atomic_rmw_add") (param $addr i64) (param $value i64) (result i64)
    (i64.atomic.rmw.add (local.get $addr) (local.get $value))
  )
  
  ;; Compare-and-swap operation
  (func (export "i64_atomic_rmw_cmpxchg") (param $addr i64) (param $old i64) (param $new i64) (result i64)
    (i64.atomic.rmw.cmpxchg (local.get $addr) (local.get $old) (local.get $new))
  )
)
```

### Bulk Memory Operations Template
```wat
(module
  (memory 1)
  
  ;; Memory fill operation
  (func (export "memory_fill_test") (param $dst i32) (param $val i32) (param $len i32)
    local.get $dst
    local.get $val
    local.get $len
    memory.fill
  )
  
  ;; Memory copy operation
  (func (export "memory_copy_test") (param $dst i32) (param $src i32) (param $len i32)
    local.get $dst
    local.get $src
    local.get $len
    memory.copy
  )
)
```

### Error Condition Testing Template
```wat
(module
  ;; Small memory for boundary testing
  (memory (;0;) i64 1 1)  ;; Only 64KB
  
  ;; Function that triggers out-of-bounds access
  (func (export "trigger_out_of_bounds")
    ;; Try to access 4GB address in 64KB memory
    i64.const 0x100000000
    i64.const 42
    i64.store  ;; This should trigger out-of-bounds exception
  )
  
  ;; Function that causes stack overflow
  (func (export "trigger_stack_overflow") (result i32)
    ;; Recursive call without termination condition
    call 0
    i32.const 1
    i32.add
  )
  
  ;; Function with unreachable instruction
  (func (export "trigger_unreachable")
    i32.const 1
    i32.const 0
    i32.div_u  ;; Division by zero
    unreachable
  )
)
```

### Multi-Module Testing Template
```wat
;; Module 1 (m1.wat)
(module
  (func (export "f1") (result i32) (i32.const 1))
  
  (memory $m1 1 2)
  (table $t1 0 funcref)
  (global $g1 i32 (i32.const 1))
  
  ;; Export resources with descriptive names
  (export "m1" (memory $m1))
  (export "m1_alias" (memory $m1))
  (export "t1" (table $t1))
  (export "g1" (global $g1))
)
```

### WASI Integration Template
```wat
(module
  ;; Import WASI functions
  (func $fd_read (import "wasi_snapshot_preview1" "fd_read") (param i32 i32 i32 i32) (result i32))
  (func $proc_exit (import "wasi_snapshot_preview1" "proc_exit") (param i32))
  
  ;; Test function using WASI
  (func (export "test_wasi_read")
    ;; Set up IOV structure
    i32.const 100  ;; iov_base
    i32.const 200  ;; buffer address
    i32.store
    i32.const 104  ;; iov_len
    i32.const 1    ;; read 1 byte
    i32.store
    
    ;; Call fd_read
    i32.const 0    ;; fd 0 (stdin)
    i32.const 100  ;; iov_base
    i32.const 1    ;; iov count
    i32.const 300  ;; return pointer
    call $fd_read
    drop
  )
  
  ;; Memory and required exports for WASI
  (memory (export "memory") 1)
  (global (export "__heap_base") i32 (i32.const 0x10000))
  (global (export "__data_end") i32 (i32.const 0x10000))
)
```

## File Placement and Naming Rules

### Directory Structure
```
tests/unit/[module_name]/
├── CMakeLists.txt
├── [module_name]_test.cc
├── [module_name]_common.h
└── wasm-apps/
    ├── feature_test.wat       # Descriptive name for feature
    ├── feature_test.wasm      # Compiled binary
    ├── edge_case.wat          # Edge case testing
    ├── edge_case.wasm
    ├── error_conditions.wat   # Error condition testing
    └── error_conditions.wasm
```

### File Naming Conventions
- **Feature-based**: `memory64_test.wat`, `atomic_opcodes.wat`, `simd_operations.wat`
- **Boundary testing**: `8GB_memory.wat`, `page_exceed_u32.wat`, `stack_overflow.wat`
- **Error conditions**: `malformed_module.wat`, `invalid_memory.wat`, `type_mismatch.wat`
- **Use underscores**: Separate words with underscores, not hyphens or spaces

## Integration with C++ Unit Tests

### Standard Test Integration Pattern
```cpp
class FeatureTestSuite : public testing::TestWithParam<RunningMode>
{
protected:
    bool load_wasm_file(const char *wasm_file)
    {
        wasm_file_buf = (unsigned char *)bh_read_file_to_buffer(wasm_file, &wasm_file_size);
        if (!wasm_file_buf) return false;
        
        module = wasm_runtime_load(wasm_file_buf, wasm_file_size, error_buf, sizeof(error_buf));
        return module != nullptr;
    }
    
    bool init_exec_env()
    {
        module_inst = wasm_runtime_instantiate(module, stack_size, heap_size, error_buf, sizeof(error_buf));
        if (!module_inst) return false;
        
        exec_env = wasm_runtime_create_exec_env(module_inst, stack_size);
        return exec_env != nullptr;
    }
    
    // Standard cleanup and member variables
    wasm_module_t module = nullptr;
    wasm_module_inst_t module_inst = nullptr;
    wasm_exec_env_t exec_env = nullptr;
    uint32_t stack_size = 8092, heap_size = 8092;
};

TEST_P(FeatureTestSuite, TestSpecificFeature)
{
    ASSERT_TRUE(load_wasm_file("feature_test.wasm"));
    ASSERT_TRUE(init_exec_env());
    
    // Set running mode
    RunningMode mode = GetParam();
    ASSERT_TRUE(wasm_runtime_set_running_mode(module_inst, mode));
    
    // Look up and call test function
    wasm_function_inst_t func = wasm_runtime_lookup_function(module_inst, "test_function");
    ASSERT_TRUE(func != nullptr);
    
    uint32_t wasm_argv[4];
    // Set up parameters using helper macros for i64 values
    PUT_I64_TO_ADDR(wasm_argv, 0x1000);        // address parameter
    PUT_I64_TO_ADDR(wasm_argv + 2, 0xdeadbeef); // value parameter
    
    ASSERT_TRUE(wasm_runtime_call_wasm(exec_env, func, 4, wasm_argv));
    
    // Verify results
    uint64_t result = GET_U64_FROM_ADDR(wasm_argv);
    ASSERT_EQ(0xdeadbeef, result);
}
```

### Parameter Handling for Memory64
```cpp
// Helper macros for i64 parameter handling
PUT_I64_TO_ADDR(wasm_argv, 0x100000000);     // Set 64-bit address
PUT_I64_TO_ADDR(wasm_argv + 2, 0xcafebeef);  // Set 64-bit value
uint64_t result = GET_U64_FROM_ADDR(wasm_argv); // Get 64-bit result
uint32_t i32_result = wasm_argv[0];           // Get 32-bit result
```

## Build System Integration

### CMakeLists.txt Integration
```cmake
# Copy WASM files to build directory
add_custom_command(TARGET ${test_name} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy
    ${CMAKE_CURRENT_SOURCE_DIR}/wasm-apps/*.wasm
    ${CMAKE_CURRENT_BINARY_DIR}/
    COMMENT "Copy test wasm files to the directory of google test"
)
```

### WAT Compilation Commands
```bash
# Basic compilation
wat2wasm test_module.wat -o test_module.wasm

# With Memory64 support
wat2wasm --enable-memory64 memory64_test.wat -o memory64_test.wasm

# With threads/atomic support
wat2wasm --enable-threads atomic_test.wat -o atomic_test.wasm

# With multiple features
wat2wasm --enable-memory64 --enable-threads --enable-simd full_test.wat -o full_test.wasm
```

## Documentation and Comments

### Required Comments in WAT Files
```wat
(module
  ;; Purpose: Test Memory64 operations with 8GB address space
  ;; Features: Memory64, large offsets, boundary conditions
  
  ;; Memory definition with size documentation
  ;; 8GB = 131072 pages (65536 bytes per page)
  (memory (;0;) i64 131072 131072)
  
  ;; Test function: Load with 4GB+ offset
  ;; Parameters: $addr (i64) - base address for load operation
  ;; Returns: i64 value loaded from addr + 4GB offset
  (func (export "i64_load_offset_4GB") (param $addr i64) (result i64)
    ;; Load from base address + 4GB offset (0x100000000)
    (i64.load offset=0x100000000 (local.get $addr))
  )
)
```

## Quality Assurance Rules

### Validation Checklist
1. **WAT Syntax**: Verify WAT compiles without errors using `wat2wasm`
2. **Module Loading**: Ensure module loads correctly in WAMR
3. **Function Exports**: Validate all expected functions are exported
4. **Parameter Types**: Confirm parameter and return types match test expectations
5. **Memory Layout**: Verify memory declarations support test requirements
6. **Feature Flags**: Ensure compilation uses correct WebAssembly feature flags

### Common Pitfalls to Avoid
1. **Memory Size Errors**: Don't use i32 addressing for Memory64 tests
2. **Missing Shared**: Atomic operations require `shared` memory declaration
3. **Wrong Offsets**: Large offsets must be within memory bounds
4. **Type Mismatches**: Parameter types must match function signatures
5. **Missing Exports**: Test functions must be exported to be callable from C++

## Version Control Guidelines

### What to Commit
- **WAT source files**: Always commit the authoritative WAT source
- **WASM binary files**: Commit compiled binaries for reproducibility
- **Documentation**: Include comments explaining test purpose and setup

### What NOT to Commit
- **Temporary compilation artifacts**: Intermediate files from wat2wasm
- **Build-generated files**: Files created by CMake during build process
- **Editor backup files**: Temporary files created by text editors

This comprehensive guide ensures consistent, maintainable, and effective WAT file generation for WAMR unit testing across all scenarios and use cases.