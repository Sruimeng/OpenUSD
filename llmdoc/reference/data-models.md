---
id: data-models
type: reference
related_ids: [tech-stack, constitution]
---

# Data Models Reference

## Purpose

Defines the core data types, value containers, and scene description primitives in OpenUSD. This is the type system foundation for all USD operations.

## Type System Hierarchy

```
Application Layer
    |
    v
UsdPrim (pxr/usd/usd/)
    |
    v
SdfPrimSpec (pxr/usd/sdf/)
    |
    v
VtValue (pxr/base/vt/)
    |
    v
Gf* Types (pxr/base/gf/)
```

## 1. Core Math Types (pxr/base/gf/)

### 1.1 Vector Types

| Type | Precision | Dimensions | Size (bytes) | Use Case |
|------|-----------|------------|--------------|----------|
| GfVec2f | float | 2D | 8 | UV coordinates, 2D positions |
| GfVec2d | double | 2D | 16 | High-precision 2D |
| GfVec2h | half | 2D | 4 | GPU-optimized 2D |
| GfVec2i | int | 2D | 8 | Integer indices |
| GfVec3f | float | 3D | 12 | Positions, normals, colors |
| GfVec3d | double | 3D | 24 | High-precision 3D |
| GfVec3h | half | 3D | 6 | GPU-optimized 3D |
| GfVec3i | int | 3D | 12 | Integer 3D indices |
| GfVec4f | float | 4D | 16 | RGBA, homogeneous coords |
| GfVec4d | double | 4D | 32 | High-precision 4D |
| GfVec4h | half | 4D | 8 | GPU-optimized 4D |
| GfVec4i | int | 4D | 16 | Integer 4D indices |

**Type Definition Pattern:**
```cpp
// See: pxr/base/gf/vec3f.h
class GfVec3f {
    float _data[3];  // Storage: contiguous array
public:
    GfVec3f(float x, float y, float z);
    float operator[](size_t i) const;
    GfVec3f operator+(const GfVec3f& rhs) const;
    float GetLength() const;
    GfVec3f GetNormalized() const;
};
```

### 1.2 Matrix Types

| Type | Size | Storage Order | Size (bytes) | Use Case |
|------|------|---------------|--------------|----------|
| GfMatrix2f | 2x2 | Row-major | 16 | 2D transforms |
| GfMatrix2d | 2x2 | Row-major | 32 | High-precision 2D |
| GfMatrix3f | 3x3 | Row-major | 36 | 3D rotations, scales |
| GfMatrix3d | 3x3 | Row-major | 72 | High-precision 3D |
| GfMatrix4f | 4x4 | Row-major | 64 | 3D transforms (GPU) |
| GfMatrix4d | 4x4 | Row-major | 128 | 3D transforms (CPU) |

**CRITICAL: Storage Order**
```cpp
// See: pxr/base/gf/matrix4d.h:48-69
class GfMatrix4d {
    double _mtx[4][4];  // Row-major: _mtx[row][col]
public:
    // Access: matrix[row][col]
    double* operator[](size_t row);

    // Transform point: result = point * matrix (right-multiply)
    GfVec3d Transform(const GfVec3d& point) const;

    // Transform direction: result = direction * matrix (no translation)
    GfVec3d TransformDir(const GfVec3d& dir) const;
};
```

**Matrix Multiplication Convention:**
```
result = point * matrix  // Right-multiply (USD convention)
NOT: result = matrix * point
```

### 1.3 Quaternion Types

| Type | Precision | Size (bytes) | Components |
|------|-----------|--------------|------------|
| GfQuatf | float | 16 | (real, i, j, k) |
| GfQuatd | double | 32 | (real, i, j, k) |
| GfQuath | half | 8 | (real, i, j, k) |
| GfDualQuatf | float | 32 | (real_quat, dual_quat) |
| GfDualQuatd | double | 64 | (real_quat, dual_quat) |
| GfDualQuath | half | 16 | (real_quat, dual_quat) |

**Type Definition:**
```cpp
// See: pxr/base/gf/quatd.h
class GfQuatd {
    double _real;      // Scalar part
    GfVec3d _imaginary; // Vector part (i, j, k)
public:
    GfQuatd(double real, double i, double j, double k);
    GfMatrix3d GetMatrix() const;  // Convert to rotation matrix
    GfQuatd GetNormalized() const;
};
```

### 1.4 Geometric Primitives

| Type | Purpose | Key Methods |
|------|---------|-------------|
| GfRange1f/d | 1D interval | GetMin(), GetMax(), Contains() |
| GfRange2f/d | 2D bounding box | GetMin(), GetMax(), GetSize() |
| GfRange3f/d | 3D bounding box | GetMin(), GetMax(), GetCorner() |
| GfBBox3d | Oriented bounding box | GetRange(), GetMatrix() |
| GfPlane | 3D plane | GetNormal(), GetDistanceFromOrigin() |
| GfLine | Infinite line | GetPoint(), GetDirection() |
| GfRay | Ray (origin + direction) | GetPoint(), Intersect() |
| GfFrustum | View frustum | ComputeViewMatrix(), Intersects() |
| GfRotation | Axis-angle rotation | GetAxis(), GetAngle(), GetQuat() |

**Range Type Pattern:**
```cpp
// See: pxr/base/gf/range3d.h
class GfRange3d {
    GfVec3d _min;
    GfVec3d _max;
public:
    GfRange3d(const GfVec3d& min, const GfVec3d& max);
    bool Contains(const GfVec3d& point) const;
    GfRange3d GetUnion(const GfRange3d& other) const;
    GfVec3d GetSize() const { return _max - _min; }
};
```

## 2. Value Container Types (pxr/base/vt/)

### 2.1 VtValue - Type-Erased Container

```cpp
// See: pxr/base/vt/value.h
class VtValue {
    // Type-erased storage (holds any copyable type)
public:
    template<typename T>
    VtValue(const T& value);  // Construct from any type

    template<typename T>
    bool IsHolding() const;   // Check held type

    template<typename T>
    T Get() const;            // Extract value (throws if wrong type)

    template<typename T>
    bool IsArrayValued() const;  // Check if holding VtArray<T>
};
```

**Usage Pattern:**
```cpp
VtValue v1 = 42;                    // Holds int
VtValue v2 = GfVec3f(1, 2, 3);      // Holds GfVec3f
VtValue v3 = VtVec3fArray({...});   // Holds array

if (v2.IsHolding<GfVec3f>()) {
    GfVec3f vec = v2.Get<GfVec3f>();
}
```

### 2.2 VtArray<T> - Typed Array Container

```cpp
// See: pxr/base/vt/array.h
template<typename T>
class VtArray {
    // Copy-on-write array with shared storage
public:
    VtArray();
    VtArray(size_t size);

    size_t size() const;
    T& operator[](size_t i);
    const T& operator[](size_t i) const;

    void push_back(const T& value);
    void resize(size_t newSize);
};
```

**Common Type Aliases:**
```cpp
// See: pxr/base/vt/types.h
using VtBoolArray   = VtArray<bool>;
using VtIntArray    = VtArray<int>;
using VtFloatArray  = VtArray<float>;
using VtDoubleArray = VtArray<double>;
using VtStringArray = VtArray<std::string>;

using VtVec2fArray  = VtArray<GfVec2f>;
using VtVec3fArray  = VtArray<GfVec3f>;
using VtVec4fArray  = VtArray<GfVec4f>;

using VtMatrix4dArray = VtArray<GfMatrix4d>;
using VtQuatfArray    = VtArray<GfQuatf>;
```

## 3. Scene Description Types (pxr/usd/sdf/)

### 3.1 SdfPath - Hierarchical Path

```cpp
// See: pxr/usd/sdf/path.h
class SdfPath {
    // Immutable, interned path string
public:
    SdfPath(const std::string& path);  // "/World/Cube.points"

    bool IsAbsolutePath() const;       // Starts with '/'
    bool IsPrimPath() const;           // "/World/Cube"
    bool IsPropertyPath() const;       // "/World/Cube.points"

    SdfPath GetParentPath() const;     // "/World/Cube" -> "/World"
    std::string GetName() const;       // "/World/Cube" -> "Cube"
    SdfPath AppendChild(const TfToken& name) const;
    SdfPath AppendProperty(const TfToken& name) const;
};
```

**Path Syntax:**
```
/World                    # Prim path
/World/Cube               # Child prim
/World/Cube.points        # Attribute
/World/Cube.material      # Relationship
/World/Cube{variant=sel}  # Variant selection
```

### 3.2 SdfLayer - File/Layer Container

```cpp
// See: pxr/usd/sdf/layer.h
class SdfLayer {
    // Single USD file (usda/usdc/usdz)
public:
    static SdfLayerRefPtr CreateNew(const std::string& identifier);
    static SdfLayerRefPtr FindOrOpen(const std::string& identifier);

    SdfPrimSpecHandle GetPrimAtPath(const SdfPath& path);
    bool HasSpec(const SdfPath& path) const;

    bool Save();
    bool Export(const std::string& filename);
};
```

### 3.3 SdfPrimSpec - Prim Description

```cpp
// See: pxr/usd/sdf/primSpec.h
class SdfPrimSpec {
public:
    SdfSpecifier GetSpecifier() const;  // Def/Over/Class
    TfToken GetTypeName() const;        // "Mesh", "Xform", etc.

    SdfAttributeSpecHandle CreateAttribute(
        const TfToken& name,
        const SdfValueTypeName& typeName
    );

    SdfRelationshipSpecHandle CreateRelationship(
        const TfToken& name
    );
};
```

### 3.4 SdfAttributeSpec - Typed Data Container

```cpp
// See: pxr/usd/sdf/attributeSpec.h
class SdfAttributeSpec {
public:
    SdfValueTypeName GetTypeName() const;  // "float3", "matrix4d", etc.
    VtValue GetDefaultValue() const;
    void SetDefaultValue(const VtValue& value);

    bool IsCustom() const;
    SdfVariability GetVariability() const;  // Uniform/Varying
};
```

### 3.5 SdfRelationshipSpec - Connections

```cpp
// See: pxr/usd/sdf/relationshipSpec.h
class SdfRelationshipSpec {
public:
    SdfPathVector GetTargetPathList() const;
    void SetTargetPathList(const SdfPathVector& paths);

    bool IsCustom() const;
};
```

## 4. Specifier Types

```cpp
// See: pxr/usd/sdf/types.h
enum SdfSpecifier {
    SdfSpecifierDef,    // Concrete prim definition (creates new prim)
    SdfSpecifierOver,   // Override existing prim (sparse edits)
    SdfSpecifierClass   // Abstract class (not instantiated)
};
```

**Usage Rules:**
```
Def:   Use for NEW prims (e.g., "def Mesh 'Cube'")
Over:  Use for EDITING existing prims (e.g., "over 'Cube'")
Class: Use for REUSABLE templates (e.g., "class '_MaterialBase'")
```

## 5. Value Type Names

| SdfValueTypeName | C++ Type | VtArray Alias | Use Case |
|------------------|----------|---------------|----------|
| `bool` | bool | VtBoolArray | Flags |
| `int` | int | VtIntArray | Indices |
| `float` | float | VtFloatArray | Scalars |
| `double` | double | VtDoubleArray | High-precision scalars |
| `string` | std::string | VtStringArray | Text |
| `token` | TfToken | VtTokenArray | Identifiers |
| `float2` | GfVec2f | VtVec2fArray | UV coords |
| `float3` | GfVec3f | VtVec3fArray | Positions, normals |
| `float4` | GfVec4f | VtVec4fArray | Colors (RGBA) |
| `double3` | GfVec3d | VtVec3dArray | High-precision 3D |
| `matrix4d` | GfMatrix4d | VtMatrix4dArray | Transforms |
| `quatf` | GfQuatf | VtQuatfArray | Rotations |
| `frame4d` | GfMatrix4d | - | Transform frames |

## 6. Data Flow Architecture

```
┌─────────────────┐
│  Application    │
│  (Python/C++)   │
└────────┬────────┘
         │
         v
┌─────────────────┐
│    UsdPrim      │  High-level API
│  (pxr/usd/usd)  │  - GetAttribute()
└────────┬────────┘  - CreateAttribute()
         │
         v
┌─────────────────┐
│  SdfPrimSpec    │  Scene description
│  (pxr/usd/sdf)  │  - Layer composition
└────────┬────────┘  - Path resolution
         │
         v
┌─────────────────┐
│    VtValue      │  Type-erased container
│  (pxr/base/vt)  │  - Runtime type checking
└────────┬────────┘  - Copy-on-write arrays
         │
         v
┌─────────────────┐
│   Gf* Types     │  Math primitives
│  (pxr/base/gf)  │  - Vectors, matrices
└─────────────────┘  - Geometric types
```

## 7. Memory Model

### 7.1 Copy-on-Write (CoW)

```cpp
// VtArray uses CoW for efficiency
VtVec3fArray a = {...};
VtVec3fArray b = a;  // Shallow copy (shares storage)

b[0] = GfVec3f(1, 2, 3);  // Triggers deep copy (detaches storage)
```

### 7.2 Reference Counting

```cpp
// Sdf types use intrusive reference counting
SdfLayerRefPtr layer = SdfLayer::CreateNew("test.usda");
// layer is automatically deleted when last RefPtr goes out of scope
```

## 8. Type Registration

```cpp
// See: pxr/base/tf/type.h
// All USD types are registered with TfType system
TfType type = TfType::Find<GfVec3f>();
if (type.IsA<GfVec3f>()) {
    // Type introspection
}
```

## Constraints

- DO NOT use raw pointers for Sdf types (use SdfHandle/RefPtr)
- DO NOT assume matrix storage order (always use USD's row-major convention)
- DO NOT mix precision types (float vs double) without explicit conversion
- DO NOT modify VtArray elements without checking if detached (CoW)
- DO NOT create SdfPath from non-validated strings (use SdfPath::IsValidPathString)
- DO NOT assume VtValue holds expected type (always check with IsHolding<T>)
- DO NOT use GfVec* for large arrays (use VtArray<GfVec*> instead)

## Related Files

- pxr/base/gf/vec3f.h - Vector type definitions
- pxr/base/gf/matrix4d.h - Matrix type definitions
- pxr/base/vt/value.h - Type-erased value container
- pxr/base/vt/array.h - Copy-on-write array
- pxr/usd/sdf/path.h - Hierarchical path system
- pxr/usd/sdf/layer.h - Layer/file container
- pxr/usd/sdf/primSpec.h - Prim specification
- pxr/usd/sdf/attributeSpec.h - Attribute specification
