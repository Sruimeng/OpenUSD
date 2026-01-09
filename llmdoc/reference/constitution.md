---
id: constitution
type: reference
related_ids: [tech-stack, data-models]
---

# OpenUSD Constitution

## Purpose

This document defines the immutable rules and conventions for OpenUSD development. All code MUST adhere to these standards.

## 1. Coordinate System Convention

| Aspect | Value | Rationale |
|--------|-------|-----------|
| Up Axis | +Y | Industry standard for 3D graphics |
| Forward Axis | -Z | Right-handed coordinate system |
| Handedness | Right-handed | Consistent with OpenGL/Vulkan |

**Source:** `pxr/base/gf/camera.h:573-575`, `pxr/base/gf/frustum.h:45-46`

```cpp
// Camera looks down -Z axis with +Y up
// Right-handed coordinate system
```

## 2. Matrix Order Convention

| Property | Value | Storage Layout |
|----------|-------|----------------|
| Storage Order | Row-Major | `[m00, m01, m02, m03, m10, m11, ...]` |
| Vector Convention | Row Vectors | `v * M` (vector on left) |
| Translation Location | Last Row | `m[3][0], m[3][1], m[3][2]` |

**Source:** `pxr/base/gf/matrix4d.h:48-69`

```cpp
// Matrix layout (row-major):
// [ m00  m01  m02  m03 ]
// [ m10  m11  m12  m13 ]
// [ m20  m21  m22  m23 ]
// [ tx   ty   tz   tw  ]  <- Translation in last row
```

**Transform Composition:**
```
result = vector * Matrix1 * Matrix2 * Matrix3
```

## 3. Precision Standards

| Type | Use Case | Epsilon Constant |
|------|----------|------------------|
| `double` | Default precision for transforms | `GF_MIN_VECTOR_LENGTH` |
| `float` | GPU-optimized data | `ARCH_MIN_FLOAT_EPS_SQR` |
| `half` | Storage optimization | N/A |

**Comparison Function:**
```cpp
// ALWAYS use GfIsClose for floating-point comparisons
bool GfIsClose(double a, double b, double epsilon);
```

**Multi-Precision Support:**
- `GfMatrix4d` (double)
- `GfMatrix4f` (float)
- `GfVec3d`, `GfVec3f`, `GfVec3h`

## 4. Naming Conventions

| Prefix | Domain | Examples |
|--------|--------|----------|
| `Gf*` | Graphics Foundation | `GfMatrix4d`, `GfVec3f`, `GfCamera` |
| `Tf*` | Type Foundation | `TfRefPtr`, `TfToken`, `TfType` |
| `Usd*` | USD Core | `UsdStage`, `UsdPrim`, `UsdAttribute` |
| `Sdf*` | Scene Description | `SdfPath`, `SdfLayer`, `SdfSpec` |
| `Hd*` | Hydra (Imaging) | `HdRenderIndex`, `HdSceneDelegate` |
| `Vt*` | Value Types | `VtArray`, `VtValue`, `VtDictionary` |

**Case Convention:**
- Classes/Types: `CamelCase`
- Methods: `CamelCase`
- Constants: `UPPER_SNAKE_CASE`
- Macros: `TF_UPPER_SNAKE_CASE`

## 5. Memory Management

| Pattern | Type | Usage |
|---------|------|-------|
| Reference Counting | `TfRefPtr<T>` | Ownership transfer |
| Weak References | `TfWeakPtr<T>` | Non-owning observation |
| Declaration Macros | `TF_DECLARE_REF_PTRS(ClassName)` | Auto-generate typedefs |

**Pseudocode:**
```cpp
// Ownership
TfRefPtr<UsdStage> stage = UsdStage::Open(path);

// Weak observation
TfWeakPtr<UsdStage> weakStage = stage;
IF weakStage.IsExpired():
  HANDLE_EXPIRED_REFERENCE()

// Declaration
TF_DECLARE_REF_PTRS(MyClass)
// Generates: MyClassRefPtr, MyClassConstRefPtr
```

## 6. Error Handling

| Macro | Severity | Use Case |
|-------|----------|----------|
| `TF_CODING_ERROR` | Programming Error | Invalid API usage, precondition violations |
| `TF_RUNTIME_ERROR` | Runtime Error | File I/O failures, invalid data |
| `TF_WARN` | Warning | Deprecated usage, performance issues |
| `TF_FATAL_ERROR` | Fatal | Unrecoverable errors (terminates process) |

**Pattern:**
```cpp
IF NOT precondition:
  TF_CODING_ERROR("Precondition failed: %s", details)
  RETURN default_value

IF file_operation_failed:
  TF_RUNTIME_ERROR("Failed to load: %s", path)
  RETURN nullptr
```

## 7. Threading and Parallelism

| Component | Purpose | Pattern |
|-----------|---------|---------|
| Intel TBB | Parallel execution | `tbb::parallel_for` |
| `WorkParallelForN` | USD parallel loops | `WorkParallelForN(count, callback)` |
| `TfErrorTransport` | Error collection | Capture errors from parallel contexts |

**Parallel Loop Pattern:**
```cpp
WorkParallelForN(itemCount, [&](size_t begin, size_t end) {
  TfErrorMark mark;
  FOR i IN [begin, end):
    ProcessItem(items[i])

  IF mark.IsClean():
    CONTINUE
  ELSE:
    COLLECT_ERRORS(mark)
})
```

## 8. Type System

| Component | Purpose | Example |
|-----------|---------|---------|
| `TfType` | Runtime type info | `TfType::Find<UsdPrim>()` |
| `TfToken` | Interned strings | `TfToken("xformOp:translate")` |
| `VtValue` | Type-erased values | `VtValue(42)`, `VtValue(GfVec3d())` |

**Token Usage:**
```cpp
// ALWAYS use TfToken for repeated strings
STATIC CONST TfToken translateToken("xformOp:translate")

// NOT: attribute.GetName() == "xformOp:translate"
// YES: attribute.GetName() == translateToken
```

## 9. Constraints (Forbidden Patterns)

### Memory Management
- DO NOT use raw pointers for ownership
- DO NOT use `new`/`delete` directly (use `TfRefPtr` or smart pointers)
- DO NOT store raw pointers to USD objects (use `TfWeakPtr`)

### Matrix Operations
- DO NOT assume column-major matrices
- DO NOT use `vector * transpose(M)` patterns
- DO NOT hardcode matrix indices without comments

### Coordinate Systems
- DO NOT assume Z-up (always Y-up)
- DO NOT flip handedness without explicit conversion
- DO NOT mix coordinate system conventions

### Platform Assumptions
- DO NOT use 32-bit builds (64-bit only)
- DO NOT use Python 2 (Python 3.6+ required)
- DO NOT assume little-endian byte order

### String Handling
- DO NOT use `std::string` for repeated identifiers (use `TfToken`)
- DO NOT compare strings directly (use `TfToken` equality)
- DO NOT construct `TfToken` in hot loops

### Threading
- DO NOT use raw threads (use TBB or `WorkDispatcher`)
- DO NOT access USD stage from multiple threads without synchronization
- DO NOT ignore `TfErrorMark` in parallel contexts

### Error Handling
- DO NOT use exceptions for control flow
- DO NOT ignore `TF_CODING_ERROR` in production
- DO NOT use `assert()` (use `TF_AXIOM` or `TF_VERIFY`)

## 10. Build System

| Tool | Purpose | Configuration |
|------|---------|---------------|
| CMake | Build configuration | `pxrConfig.cmake` |
| Python | Build scripts | `build_scripts/build_usd.py` |
| Boost | C++ utilities | Version 1.70+ |

**Required Flags:**
```cmake
# 64-bit only
CMAKE_SIZEOF_VOID_P == 8

# C++17 minimum
CMAKE_CXX_STANDARD >= 17
```

## 11. Versioning and Compatibility

| Layer | Versioning Scheme | Example |
|-------|-------------------|---------|
| USD File Format | `#usda 1.0` | Scene description version |
| API | Semantic versioning | `24.08` (Year.Month) |
| Schema | `USDGEOM_VERSION` | Schema definition version |

**Compatibility Rule:**
```
IF file_version > library_version:
  TF_WARN("File may contain unsupported features")
  ATTEMPT_BEST_EFFORT_LOAD()
```

## 12. Performance Guidelines

| Pattern | Guideline | Rationale |
|---------|-----------|-----------|
| Stage Traversal | Use `UsdPrimRange` | Optimized iteration |
| Attribute Access | Cache `UsdAttribute` | Avoid repeated lookups |
| Token Construction | Use static `TfToken` | Avoid runtime interning |
| Parallel Iteration | Use `WorkParallelForN` | Leverage multi-core |

**Anti-Pattern:**
```cpp
// BAD: Repeated string lookups
FOR prim IN stage.Traverse():
  attr = prim.GetAttribute("points")  // Repeated TfToken construction

// GOOD: Cache token
STATIC CONST TfToken pointsToken("points")
FOR prim IN stage.Traverse():
  attr = prim.GetAttribute(pointsToken)
```

## 13. Schema Extension

| Component | Purpose | File Location |
|-----------|---------|---------------|
| `schema.usda` | Schema definition | `<plugin>/schema.usda` |
| `generatedSchema.usda` | Generated code | Auto-generated |
| `plugInfo.json` | Plugin metadata | `<plugin>/plugInfo.json` |

**Extension Pattern:**
```
DEFINE custom schema:
  1. CREATE schema.usda with type definitions
  2. RUN usdGenSchema to generate C++ code
  3. REGISTER plugin via plugInfo.json
  4. IMPLEMENT custom behavior in C++
```

## Enforcement

This constitution is enforced through:
1. Code review (human validation)
2. Static analysis (clang-tidy, custom linters)
3. Unit tests (convention validation)
4. Documentation audits (cartographer/critic agents)

**Violation Severity:**
- **Critical:** Coordinate system, matrix order violations
- **High:** Memory management, threading violations
- **Medium:** Naming convention, performance anti-patterns
- **Low:** Documentation style, comment formatting

## References

- Graphics Foundation: `pxr/base/gf/`
- Type Foundation: `pxr/base/tf/`
- USD Core: `pxr/usd/usd/`
- Scene Description: `pxr/usd/sdf/`
- Hydra Imaging: `pxr/imaging/hd/`
