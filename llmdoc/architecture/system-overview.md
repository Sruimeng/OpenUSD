---
id: system-overview
type: architecture
related_ids: [tech-stack, constitution]
---

# System Overview

## Architecture Type

**Modular Monolith with Layered Hierarchy**

OpenUSD is organized as a layered architecture where higher layers depend on lower layers, with strict dependency flow from top to bottom.

## Layer Diagram

```
┌─────────────────────────────────┐
│   Applications (usdview)        │  User-facing tools
├─────────────────────────────────┤
│   USD Imaging (usdImaging)      │  USD-to-Hydra bridge
├─────────────────────────────────┤
│   Imaging (hd, hdSt, hgi)       │  Rendering framework
├─────────────────────────────────┤
│   USD Core (usd, sdf, pcp)      │  Scene composition
├─────────────────────────────────┤
│   Base (tf, gf, vt, arch)       │  Foundation libraries
└─────────────────────────────────┘
```

## Core Modules

| Layer | Module | Purpose | Key Headers |
|-------|--------|---------|-------------|
| base | arch | Platform abstraction, system utilities | pxr/base/arch/*.h |
| base | tf | Type foundation, diagnostics, logging | pxr/base/tf/*.h |
| base | gf | Math library (vectors, matrices, transforms) | pxr/base/gf/*.h |
| base | vt | Value types, arrays | pxr/base/vt/*.h |
| base | work | Threading, task dispatch | pxr/base/work/*.h |
| base | trace | Performance profiling | pxr/base/trace/*.h |
| usd | sdf | Scene description format, low-level data | pxr/usd/sdf/*.h |
| usd | pcp | Prim composition protocol (layering, references) | pxr/usd/pcp/*.h |
| usd | usd | Core USD API (Stage, Prim, Attribute) | pxr/usd/usd/*.h |
| usd | usdGeom | Geometry schemas | pxr/usd/usdGeom/*.h |
| usd | usdShade | Shading schemas | pxr/usd/usdShade/*.h |
| usd | usdSkel | Skeletal animation | pxr/usd/usdSkel/*.h |
| imaging | hgi | Hardware Graphics Interface (GPU abstraction) | pxr/imaging/hgi/*.h |
| imaging | hd | Hydra core (render delegate protocol) | pxr/imaging/hd/*.h |
| imaging | hdSt | Storm renderer (OpenGL/Metal/Vulkan) | pxr/imaging/hdSt/*.h |
| usdImaging | usdImaging | USD scene delegate for Hydra | pxr/usdImaging/usdImaging/*.h |

## Plugin System

### Discovery Mechanism

```
FUNCTION PluginDiscovery():
  FOR EACH directory IN PXR_PLUGINPATH_NAME:
    FIND plugInfo.json files
    PARSE plugin metadata
    REGISTER plugin types
```

### Key Components

| Component | Purpose | Location |
|-----------|---------|----------|
| Plug | Plugin discovery and loading | pxr/base/plug/ |
| HdRendererPluginRegistry | Render delegate registration | pxr/imaging/hd/rendererPluginRegistry.h |
| plugInfo.json | Plugin metadata descriptor | */plugInfo.json |
| TfType | Runtime type system | pxr/base/tf/type.h |

### Plugin Types

- **Render Delegates**: hdSt (Storm), hdPrman (RenderMan)
- **File Format Plugins**: .usd, .usda, .usdc, .usdz
- **Schema Plugins**: Custom USD schemas
- **Imaging Plugins**: Scene delegates, render settings

## Entry Points

### C++ API

```cpp
// Primary headers
#include <pxr/usd/usd/stage.h>      // UsdStage: Scene container
#include <pxr/usd/usd/prim.h>       // UsdPrim: Scene object
#include <pxr/usd/usd/attribute.h>  // UsdAttribute: Property
#include <pxr/usd/usdGeom/mesh.h>   // UsdGeomMesh: Geometry

// Usage pattern
UsdStageRefPtr stage = UsdStage::Open("scene.usd");
UsdPrim prim = stage->GetPrimAtPath(SdfPath("/World/Mesh"));
UsdGeomMesh mesh(prim);
```

### Python API

```python
# Primary modules
from pxr import Usd, UsdGeom, Sdf, Gf

# Usage pattern
stage = Usd.Stage.Open("scene.usd")
prim = stage.GetPrimAtPath("/World/Mesh")
mesh = UsdGeom.Mesh(prim)
```

### CLI Tools

| Tool | Purpose | Location |
|------|---------|----------|
| usdcat | Inspect/convert USD files | pxr/usd/bin/usdcat |
| usdview | Interactive viewer | pxr/usdImaging/bin/usdview |
| usddiff | Compare USD files | pxr/usd/bin/usddiff |
| usdedit | Edit USD files | pxr/usd/bin/usdedit |
| usdchecker | Validate USD files | pxr/usd/bin/usdchecker |

## Directory Structure

```
pxr/
├── base/              # Foundation layer
│   ├── arch/          # Platform abstraction
│   ├── tf/            # Type foundation
│   ├── gf/            # Math library
│   ├── vt/            # Value types
│   ├── work/          # Threading
│   ├── trace/         # Profiling
│   └── plug/          # Plugin system
│
├── usd/               # USD Core layer
│   ├── sdf/           # Scene description
│   ├── pcp/           # Composition
│   ├── usd/           # Core API
│   ├── usdGeom/       # Geometry schemas
│   ├── usdShade/      # Shading schemas
│   ├── usdSkel/       # Skeletal animation
│   └── usdUtils/      # Utilities
│
├── imaging/           # Rendering layer
│   ├── hgi/           # GPU abstraction
│   ├── hd/            # Hydra core
│   ├── hdSt/          # Storm renderer
│   ├── hdx/           # Hydra extensions
│   └── glf/           # OpenGL foundation
│
├── usdImaging/        # Bridge layer
│   └── usdImaging/    # USD scene delegate
│
└── extras/            # Optional components
    └── usd/
        └── examples/  # Sample code
```

## Data Flow

### Scene Loading

```
FUNCTION LoadScene(filepath):
  stage = UsdStage::Open(filepath)
    -> SdfLayer::FindOrOpen(filepath)
      -> FileFormat plugin reads binary/ASCII
      -> SdfData populated
    -> PcpCache computes composition
      -> Resolve references, payloads, variants
      -> Build prim index graph
    -> UsdStage wraps composed scene
  RETURN stage
```

### Rendering Pipeline

```
FUNCTION RenderScene(stage):
  sceneDelegate = UsdImagingDelegate(stage)
    -> Populate Hydra prims from USD prims
    -> Sync geometry, materials, transforms

  renderDelegate = HdStRenderDelegate()
    -> Create GPU resources via HGI
    -> Compile shaders

  renderIndex = HdRenderIndex(renderDelegate)
    -> Track scene state
    -> Manage change tracking

  engine = HdEngine()
    -> Execute render tasks
    -> Submit draw calls
```

## Composition Model

### Layer Stack

```
STRUCTURE LayerStack:
  rootLayer: SdfLayerRefPtr
  sessionLayer: SdfLayerRefPtr (optional, strongest)
  subLayers: [SdfLayerRefPtr] (ordered by strength)

FUNCTION ResolveAttribute(path, attrName):
  FOR layer IN LayerStack (strongest to weakest):
    IF layer.HasAttribute(path, attrName):
      RETURN layer.GetAttribute(path, attrName)
  RETURN default_value
```

### Composition Arcs

| Arc Type | Strength | Purpose |
|----------|----------|---------|
| SubLayers | Strongest | Layer stacking |
| References | Strong | Asset reuse |
| Payloads | Strong | Deferred loading |
| Inherits | Medium | Class inheritance |
| Variants | Medium | Switchable variations |
| Specializes | Weakest | Override base definitions |

## Threading Model

```
STRUCTURE WorkDispatcher:
  threadPool: [Thread]
  taskQueue: ConcurrentQueue<Task>

FUNCTION ParallelFor(items, func):
  FOR EACH chunk IN Partition(items, threadCount):
    WorkDispatcher.Run(lambda:
      FOR item IN chunk:
        func(item)
    )
  WorkDispatcher.Wait()
```

## Constraints

- DO NOT bypass layer composition (always use UsdStage API)
- DO NOT modify SdfLayer directly during stage traversal
- DO NOT assume single-threaded access to UsdStage
- DO NOT hold strong references to UsdPrim (use SdfPath instead)
- DO NOT create circular references in composition arcs
- DO NOT load payloads synchronously on main thread
- DO NOT assume matrix multiplication order (see constitution.md)
- DO NOT mix coordinate systems without explicit transforms

## Related Documents

- `llmdoc/reference/constitution.md`: Matrix order, coordinate systems
- `llmdoc/reference/tech-stack.md`: Build system, dependencies
- `llmdoc/architecture/data-models.md`: USD schema definitions
