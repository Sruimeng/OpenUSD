---
id: shared-utilities
type: reference
related_ids: [tech-stack, constitution]
---

# Shared Utilities Reference

## Purpose

Cross-cutting utilities used throughout OpenUSD. These are the standard library primitives that replace raw C++ equivalents.

## Memory & Alignment

**Location:** `pxr/base/arch/align.h`

### API

| Function | Signature | Purpose |
|----------|-----------|---------|
| `ArchAlignMemorySize` | `size_t ArchAlignMemorySize(size_t nBytes)` | Round size to 8-byte boundary |
| `ArchAlignedAlloc` | `void* ArchAlignedAlloc(size_t alignment, size_t size)` | Allocate aligned memory |
| `ArchAlignedFree` | `void ArchAlignedFree(void* ptr)` | Free aligned memory |

### Constants

```cpp
ARCH_CACHE_LINE_SIZE  // 64 bytes (x86/ARM) or 128 bytes (PowerPC)
```

### Usage Pattern

```cpp
// Allocate cache-aligned buffer
size_t size = ArchAlignMemorySize(dataSize);
void* buffer = ArchAlignedAlloc(ARCH_CACHE_LINE_SIZE, size);
// ... use buffer ...
ArchAlignedFree(buffer);
```

## Hashing

### Arch Layer (Low-Level)

**Location:** `pxr/base/arch/hash.h`

| Function | Signature | Output |
|----------|-----------|--------|
| `ArchHash` | `uint32_t ArchHash(const char* data, size_t len)` | 32-bit hash |
| `ArchHash64` | `uint64_t ArchHash64(const char* data, size_t len)` | 64-bit hash |

### Tf Layer (Generic)

**Location:** `pxr/base/tf/hash.h`

```cpp
// Hash single object
TfHash hasher;
size_t h = hasher(myObject);

// Combine multiple hashes
size_t combined = TfHash::Combine(hash1, hash2, hash3);
```

### Specialization Pattern

```cpp
namespace std {
    template<>
    struct hash<MyType> {
        size_t operator()(const MyType& obj) const {
            return TfHash::Combine(
                TfHash()(obj.field1),
                TfHash()(obj.field2)
            );
        }
    };
}
```

## Math Primitives

**Location:** `pxr/base/arch/math.h`

### Functions

| Function | Signature | Behavior |
|----------|-----------|----------|
| `ArchSign` | `long ArchSign(long val)` | Returns -1, 0, or 1 |
| `ArchSinCosf` | `void ArchSinCosf(float v, float* s, float* c)` | Compute sin/cos simultaneously |
| `ArchCountTrailingZeros` | `uint32_t ArchCountTrailingZeros(uint64_t x)` | Count trailing zero bits |

### Constants

```cpp
ARCH_MIN_FLOAT_EPS_SQR  // Minimum epsilon for float comparisons (squared)
```

### Usage

```cpp
// Fast sign extraction
long sign = ArchSign(value);  // -1, 0, or 1

// Efficient sin/cos
float s, c;
ArchSinCosf(angle, &s, &c);  // Single call, optimized
```

## Error Handling

**Location:** `pxr/base/tf/diagnostic.h`

### Macros

| Macro | Severity | Use Case | Terminates? |
|-------|----------|----------|-------------|
| `TF_CODING_ERROR` | Error | Programming bugs (precondition violations) | No |
| `TF_RUNTIME_ERROR` | Error | Runtime failures (file not found) | No |
| `TF_WARN` | Warning | Recoverable issues | No |
| `TF_FATAL_ERROR` | Fatal | Unrecoverable errors | Yes |
| `TF_STATUS` | Info | Status messages | No |

### Usage Pattern

```cpp
// Precondition check
if (!prim.IsValid()) {
    TF_CODING_ERROR("Invalid prim passed to %s", __FUNCTION__);
    return false;
}

// Runtime failure
if (!file.Open()) {
    TF_RUNTIME_ERROR("Failed to open file: %s", path.c_str());
    return nullptr;
}

// Warning
if (cache.IsFull()) {
    TF_WARN("Cache full, evicting oldest entry");
}
```

### Error Delegates

```cpp
// Custom error handler
class MyErrorDelegate : public TfDiagnosticMgr::Delegate {
    void IssueError(TfError const &err) override {
        // Custom logging
    }
};

TfDiagnosticMgr::GetInstance().AddDelegate(new MyErrorDelegate);
```

## Threading

**Location:** `pxr/base/work/`

### Parallel Loops

| Function | Signature | Use Case |
|----------|-----------|----------|
| `WorkParallelForN` | `void WorkParallelForN(size_t n, Fn&& fn, size_t grainSize = 1)` | Parallel iteration |
| `WorkSerialForN` | `void WorkSerialForN(size_t n, Fn&& fn)` | Serial (debug) iteration |

```cpp
// Parallel loop
WorkParallelForN(items.size(), [&](size_t begin, size_t end) {
    for (size_t i = begin; i < end; ++i) {
        ProcessItem(items[i]);
    }
});

// Serial version (for debugging)
WorkSerialForN(items.size(), [&](size_t begin, size_t end) {
    for (size_t i = begin; i < end; ++i) {
        ProcessItem(items[i]);
    }
});
```

### Task Dispatch

```cpp
// WorkDispatcher: Coordinated tasks
WorkDispatcher dispatcher;
dispatcher.Run([&]() { Task1(); });
dispatcher.Run([&]() { Task2(); });
dispatcher.Wait();  // Wait for all tasks

// WorkDetachedTask: Fire-and-forget
WorkRunDetachedTask([=]() {
    BackgroundWork();
});
```

### Grain Size

```cpp
// Small grain size: More parallelism, higher overhead
WorkParallelForN(1000000, fn, 1);

// Large grain size: Less overhead, less parallelism
WorkParallelForN(1000000, fn, 10000);

// Rule of thumb: grainSize = totalWork / (numCores * 4)
```

## Pointer Management

**Location:** `pxr/base/tf/`

### Declaration Macros

```cpp
class MyClass;

// Weak pointers (non-owning)
TF_DECLARE_WEAK_PTRS(MyClass);
// Creates: MyClassPtr, MyClassConstPtr

// Reference-counted pointers (owning)
TF_DECLARE_REF_PTRS(MyClass);
// Creates: MyClassRefPtr, MyClassConstRefPtr
```

### Weak Pointers

```cpp
// Usage
TfWeakPtr<MyClass> weakPtr = GetObject();
if (weakPtr) {
    weakPtr->DoSomething();
}

// Expiration check
if (weakPtr.IsExpired()) {
    // Object was deleted
}
```

### Reference Pointers

```cpp
// Usage
TfRefPtr<MyClass> refPtr = TfCreateRefPtr(new MyClass);
// Automatic reference counting

// Conversion
TfWeakPtr<MyClass> weak = refPtr;  // OK
TfRefPtr<MyClass> ref = TfCreateRefPtrFromProtectedWeakPtr(weak);
```

### Implementation Requirements

```cpp
class MyClass : public TfWeakBase {
    // For TfWeakPtr support
};

class MyRefCounted : public TfRefBase {
    // For TfRefPtr support
};
```

## Tracing

**Location:** `pxr/base/trace/`

### Macros

| Macro | Scope | Purpose |
|-------|-------|---------|
| `TRACE_FUNCTION()` | Function | Trace entire function |
| `TRACE_SCOPE("name")` | Block | Trace named scope |
| `TRACE_MARKER("event")` | Point | Mark single event |
| `TRACE_COUNTER_DELTA(key, delta)` | Point | Update counter |

### Usage

```cpp
void MyFunction() {
    TRACE_FUNCTION();  // Automatic function name

    {
        TRACE_SCOPE("InitPhase");
        Initialize();
    }

    TRACE_MARKER("StartProcessing");
    Process();

    TRACE_COUNTER_DELTA("ItemsProcessed", count);
}
```

### Collection

```cpp
// Enable tracing
TraceCollector::GetInstance().SetEnabled(true);

// Collect data
TraceCollector::GetInstance().BeginEvent("MyEvent");
// ... work ...
TraceCollector::GetInstance().EndEvent("MyEvent");

// Export
TraceReporter::GetGlobalReporter()->Report(std::cout);
```

## Constraints

### DO NOT

- DO NOT use `std::hash` directly - use `TfHash` for consistency
- DO NOT use raw TBB calls - use `WorkParallelForN` for portability
- DO NOT use `assert()` - use `TF_CODING_ERROR` for better diagnostics
- DO NOT use `aligned_alloc` - use `ArchAlignedAlloc` for cross-platform support
- DO NOT use `malloc/free` for aligned memory - use `ArchAlignedAlloc/Free`
- DO NOT ignore grain size in parallel loops - tune for workload
- DO NOT use raw pointers for shared ownership - use `TfRefPtr`
- DO NOT use `std::weak_ptr` - use `TfWeakPtr` for USD objects

### DO

- DO use `TfHash::Combine` for multi-field hashing
- DO use `WorkSerialForN` for debugging parallel code
- DO use `TF_VERIFY` for runtime assertions that should not be compiled out
- DO use `TRACE_FUNCTION` in performance-critical paths
- DO check `TfWeakPtr::IsExpired()` before dereferencing
- DO use `ArchAlignMemorySize` before allocation to ensure proper alignment
- DO use `TF_CODING_ERROR` for precondition violations
- DO use `TF_RUNTIME_ERROR` for external failures

## Cross-References

- **Constitution:** See `llmdoc/reference/constitution.md` for coordinate system rules
- **Tech Stack:** See `llmdoc/reference/tech-stack.md` for dependency hierarchy
- **Data Models:** See `llmdoc/reference/data-models.md` for USD object lifecycle
