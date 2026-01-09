---
id: tech-stack
type: reference
related_ids: [constitution, infrastructure-investigation]
---

# Tech Stack

## Context

OpenUSD 技术栈定义。基于基础设施调查结果（2026-01-09）。

## Primary Languages

| Language | File Count | Purpose | Standard |
|----------|-----------|---------|----------|
| C++ | ~2,745 | Core libraries, runtime | C++17 required |
| Python | ~665 | Build scripts, bindings, tools | 3.x |
| CMake | ~14 | Build configuration | 3.26+ |

## Build System

```
ORCHESTRATOR: build_scripts/build_usd.py
  ├─ CMake 3.26+
  ├─ C++17 compiler
  └─ Python 3.x interpreter

ENTRY_POINTS:
  - build_scripts/build_usd.py (Main orchestrator)
  - CMakeLists.txt (Root configuration)
  - cmake/defaults/Options.cmake (Feature flags)
```

## Core Dependencies

### Threading
| Library | Purpose | Status |
|---------|---------|--------|
| Intel TBB | Parallel execution | Required |

### Language Bindings
| Library | Purpose | Status |
|---------|---------|--------|
| Python 3.x | Python bindings, tools | Required |
| Boost | Python interop (legacy) | Optional (for OpenVDB) |

## Graphics Stack

### Rendering APIs
| API | Platform | Status |
|-----|----------|--------|
| OpenGL | Cross-platform | Core |
| Vulkan | Linux, Windows | Core (with shaderc_combined) |
| Metal | macOS, iOS, visionOS | Core |

### Graphics Libraries
| Library | Purpose | Status |
|---------|---------|--------|
| OpenSubdiv | Subdivision surfaces | Core |
| OpenImageIO | Image I/O | Core |
| OpenColorIO | Color management | Core |
| OpenEXR | HDR images | Core |

## Optional Plugins

### Interchange
| Plugin | Purpose | Dependencies |
|--------|---------|--------------|
| Alembic | Alembic format support | HDF5 |
| MaterialX | Material definitions | MaterialX lib |

### Compression
| Plugin | Purpose | Dependencies |
|--------|---------|--------------|
| Draco | Geometry compression | Draco lib |

### Rendering
| Plugin | Purpose | Dependencies |
|--------|---------|--------------|
| Embree | Ray tracing | Intel Embree |
| RenderMan | RenderMan integration | RenderMan SDK |

## Platform Support

### Desktop
| Platform | Compiler | Notes |
|----------|----------|-------|
| Linux | GCC/Clang | CentOS 7+ baseline |
| macOS | Clang | Metal required |
| Windows | MSVC 2017/2019/2022 | VS required |

### Embedded/Web
| Platform | Toolchain | Status |
|----------|-----------|--------|
| WebAssembly | Emscripten | Experimental |
| iOS | Xcode | Supported |
| visionOS | Xcode | Supported |

## CI/CD

```
PIPELINE:
  .github/workflows/buildusd.yml
    ├─ Matrix: [Linux, macOS, Windows]
    ├─ Python: [3.7, 3.8, 3.9, 3.10, 3.11]
    └─ Configurations: [Debug, Release]

  .github/workflows/pypi.yml
    └─ Python package distribution
```

### GitHub Actions
| Workflow | Purpose | Trigger |
|----------|---------|---------|
| buildusd.yml | Multi-platform build | Push, PR |
| pypi.yml | Python package publish | Release tag |

## Build Configuration

### CMake Options
```cmake
# Core Features
PXR_BUILD_IMAGING      # Imaging libraries
PXR_BUILD_USD_IMAGING  # USD imaging
PXR_ENABLE_PYTHON_SUPPORT  # Python bindings

# Optional Features
PXR_BUILD_ALEMBIC_PLUGIN
PXR_BUILD_DRACO_PLUGIN
PXR_BUILD_EMBREE_PLUGIN
PXR_BUILD_MATERIALX_PLUGIN
PXR_BUILD_OPENCOLORIO_PLUGIN
PXR_BUILD_OPENIMAGEIO_PLUGIN
```

See: `cmake/defaults/Options.cmake`

## Constraints

- DO NOT use C++ features beyond C++17
- DO NOT assume TBB is optional (required for core)
- DO NOT mix Python 2.x and 3.x
- DO NOT use platform-specific APIs without guards
- DO NOT bypass build_usd.py for dependency management
- DO NOT commit binaries to repository

## Version Requirements

```
MINIMUM:
  CMake: 3.26
  C++: 17
  Python: 3.7
  TBB: 2017+

RECOMMENDED:
  CMake: 3.28+
  Python: 3.11+
  Compiler: Latest stable
```

## References

- Build System: `build_scripts/build_usd.py`
- CMake Root: `CMakeLists.txt`
- Options: `cmake/defaults/Options.cmake`
- CI Config: `.github/workflows/`
