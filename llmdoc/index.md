---
id: index
type: index
related_ids: [constitution, tech-stack, data-models, system-overview, shared-utilities, doc-standard]
---

# OpenUSD Documentation Index

> **LLM-Optimized Documentation System**
> Generated: 2026-01-09

## Quick Start

```
PRIORITY READ ORDER:
1. constitution.md      → Rules of Engagement (MUST READ FIRST)
2. system-overview.md   → Architecture & Layers
3. data-models.md       → Core Types & Schemas
4. tech-stack.md        → Build & Dependencies
5. shared-utilities.md  → Don't Reinvent List
```

---

## Document Map

### Reference (The Constitution)

| Document | Purpose | Priority |
|----------|---------|----------|
| [constitution.md](reference/constitution.md) | **Rules of Engagement**: Coordinate systems, matrix conventions, naming, forbidden patterns | **CRITICAL** |
| [tech-stack.md](reference/tech-stack.md) | Build system, dependencies, platforms | High |
| [data-models.md](reference/data-models.md) | Core types: Gf*, Vt*, Sdf* | High |
| [shared-utilities.md](reference/shared-utilities.md) | "Don't Reinvent" list: TfHash, WorkParallelForN, etc. | High |

### Architecture

| Document | Purpose |
|----------|---------|
| [system-overview.md](architecture/system-overview.md) | Layered architecture, module map, plugin system |

### Guides

| Document | Purpose |
|----------|---------|
| [doc-standard.md](guides/doc-standard.md) | Documentation standards for this system |

### Agent (Strategic Memory)

| Document | Purpose |
|----------|---------|
| [strategy-draco-compress.md](agent/strategy-draco-compress.md) | Draco compression strategy |
| [strategy-3d-format-comparison.md](agent/strategy-3d-format-comparison.md) | 3D format comparison analysis |

---

## Key Conventions Summary

### Coordinate System
```
Y-up, -Z-forward, Right-handed
Source: pxr/base/gf/frustum.h:45-46
```

### Matrix Order
```
Row-Major Storage, Row Vectors
Translation in LAST ROW (not column)
Source: pxr/base/gf/matrix4d.h:48-69
```

### Naming Prefixes
| Prefix | Domain |
|--------|--------|
| Gf* | Graphics Foundation (math) |
| Tf* | Type Foundation (utilities) |
| Vt* | Value Types (containers) |
| Sdf* | Scene Description Format |
| Usd* | USD Core API |
| Hd* | Hydra (rendering) |

### Critical DON'Ts
```
DO NOT assume column-major matrices
DO NOT use raw pointers for ownership
DO NOT use Python 2
DO NOT use 32-bit builds
DO NOT reinvent TfHash, WorkParallelForN
```

---

## Directory Structure

```
llmdoc/
├── index.md                    # This file
├── reference/                  # The Constitution
│   ├── constitution.md         # Rules of Engagement
│   ├── tech-stack.md           # Build & Dependencies
│   ├── data-models.md          # Core Types
│   └── shared-utilities.md     # Don't Reinvent List
├── architecture/               # System Design
│   └── system-overview.md      # Layered Architecture
├── guides/                     # How-To
│   └── doc-standard.md         # Documentation Rules
└── agent/                      # Strategic Memory
    ├── strategy-*.md           # Strategy Documents
    └── ...
```

---

## Codebase Quick Reference

### Entry Points
| Type | Location |
|------|----------|
| C++ Stage API | `pxr/usd/usd/stage.h` |
| C++ Prim API | `pxr/usd/usd/prim.h` |
| Python | `from pxr import Usd, UsdGeom` |
| CLI | `usdcat`, `usdview`, `usddiff` |

### Layer Hierarchy
```
pxr/base/      → Foundation (tf, gf, vt, arch)
pxr/usd/       → Core USD (sdf, pcp, usd, usdGeom)
pxr/imaging/   → Rendering (hgi, hd, hdSt)
pxr/usdImaging → Bridge (usdImaging, usdview)
```
