# SpatialOS

**Hierarchical navigation-mesh pathfinding in C++, built on Recast and Detour.**

This repository contains a research prototype for constructing hierarchical graphs over navigation meshes and exploring pathfinding through an interactive SDL/OpenGL application. It includes custom navigation code, graph partitioning with METIS, path-query experiments, and visualization tools.

The repository also contains Improbable SpatialOS worker headers and connection scaffolding. The connection helpers in `RecastDemo/Source/main.cpp` are commented out; the current entry point launches the local Recast demo.

## Features

- **Navigation-mesh construction:** generate walkable polygon meshes from OBJ geometry using Recast.
- **Hierarchical graph construction:** group navigation polygons using METIS and construct multiple levels of graph abstraction.
- **Portal connections:** represent connections between clusters and store intra-cluster paths and costs.
- **Pathfinding experiments:** custom hierarchical queries alongside Detour-based navigation queries.
- **Interactive inspection:** build navigation meshes and hierarchies, select start and goal positions, and inspect results in the demo.
- **Experimental logging:** write timing and path-related information to text files from selected code paths.

The bundled Recast demo also includes tiled meshes, temporary obstacles, crowd tools, and other upstream navigation utilities. These components have separate behavior from the custom hierarchical implementation.

## Approach

The custom navigation code follows this general workflow:

1. Build a navigation mesh from input geometry.
2. Represent its polygons and adjacency as a graph.
3. Partition and merge graph nodes into clusters using METIS.
4. Construct higher-level graphs with connections between clusters and paths within each cluster.
5. Connect query endpoints to the hierarchy, search the graph, and expand the result into a polygon path.

The main implementation is in [`Navigation.cpp`](HierarchicalPathfinding/Source/Navigation.cpp), with graph data structures in [`Graph.h`](HierarchicalPathfinding/Include/Graph.h). The demo invokes hierarchy construction from `Sample_SoloMesh::handleHierarchy()`.

## Repository layout

| Path | Purpose |
| --- | --- |
| `HierarchicalPathfinding/` | Custom hierarchical navigation implementation and graph structures |
| `Recast/` | Navigation-mesh generation |
| `Detour/` | Navigation-mesh queries and pathfinding |
| `DetourCrowd/` | Crowd navigation utilities |
| `DetourTileCache/` | Tile-cache and obstacle utilities |
| `DebugUtils/` | Navigation debugging and drawing helpers |
| `RecastDemo/Source/` | Interactive application, tools, and experiments |
| `RecastDemo/Include/improbable/` | SpatialOS worker and component headers |
| `RecastDemo/Contrib/fastlz/` | Compression sources and bundled METIS headers/libraries |
| `RecastDemo/premake4.lua` | Legacy Premake 4 build configuration |
| `liB/`, `Bin/`, `RecastDemo/Bin/` | Historical libraries, runtime DLLs, and build artifacts |

## Build status and requirements

**The repository requires build configuration cleanup before a reproducible build is available.** It preserves a legacy development snapshot with Windows-specific code and machine-specific include paths. No successful clean build or automated test run is documented here.

Clone the source with:

```bash
git clone https://github.com/rahmani-vahid/SpatialOS.git
cd SpatialOS
```

The source and existing build configuration reference:

| Dependency | Role |
| --- | --- |
| C++ toolchain with C++11 support | Standard threads, containers, and application code |
| Windows SDK / MSVC concurrency headers | `windows.h`, `ppl.h`, and `agents.h` in the custom navigation source |
| SDL 1.2 | Windowing, input, and OpenGL context creation |
| OpenGL, GLU, and GLUT | Rendering and graphics utilities |
| METIS | Graph partitioning; the source uses the legacy `METIS_PartGraphKway` calling convention |
| Premake 4 | Existing project-generation script |

Some libraries are committed as historical binaries. Their compiler, architecture, and ABI compatibility must be checked against the chosen toolchain.

### Build preparation

Before attempting to compile:

1. Replace absolute `D:/...` include paths in `RecastDemo/Include/Navigation.h` and the bundled METIS headers with portable includes.
2. Reconcile the duplicate `Navigation.h` and `Graph.h` headers under `RecastDemo/Include/` and `HierarchicalPathfinding/Include/`.
3. Extend the build configuration to compile and link `HierarchicalPathfinding/Source/Navigation.cpp` and the matching METIS library. The checked-in Premake file does not define a hierarchical-pathfinding project.
4. Resolve SDL, GLUT, Windows concurrency, and any retained SpatialOS SDK dependencies for the selected compiler.
5. Exclude `RecastDemo/Source/NavMeshTesterTool - backup.cpp` from compilation. The existing source wildcard also selects this backup file.
6. Supply the runtime font and mesh assets described below.

The historical project-generation entry point is:

```bash
cd RecastDemo
premake4 --help
```

After updating `premake4.lua`, select an action supported by your Premake installation and toolchain. The script contains Windows, Linux, and macOS configuration sections, but those sections do not establish that the modified application currently builds on all three platforms.

## Interactive workflow

After resolving the build requirements and producing the demo executable:

1. Set the working directory to a directory containing `DroidSans.ttf` and a `Meshes/` folder with OBJ geometry. These assets are not included in the current checkout.
2. Launch the demo and choose a sample and mesh.
3. Configure the navigation-mesh settings and select **Build**.
4. For the solo-mesh hierarchy workflow, configure **Max level**, **Level**, and **Num merged polys**, then select **Build Hierarchy**.
5. Select the navigation test tool and choose query endpoints.

| Control | Action |
| --- | --- |
| `W`, `A`, `S`, `D` | Move the camera |
| Right mouse button | Rotate the camera |
| `Shift` + left mouse button | Set the start position in the navigation test tool |
| Left mouse button | Set the goal position in the navigation test tool |

Several experimental routines write text files relative to the working directory, including append-mode logs. Inspect the relevant routine before using these files for measurements.

## SpatialOS integration

SpatialOS-related headers live under `RecastDemo/Include/improbable/`, including a pathfinding component header. `main.cpp` contains commented connection helpers for locator and receptionist connections.

Running a distributed worker would require restoring and validating that integration, supplying a compatible SDK and deployment configuration, and connecting the navigation queries to worker operations. This repository does not currently document a complete deployment workflow.

## Validation and limitations

This snapshot includes experimental timing/logging code and Recast demo test-case utilities. It does not include a standalone automated test suite, CI configuration, or a reproducible benchmark dataset and procedure.

No speedup, path-quality, optimality, or distributed-scaling result is claimed by this README. Reproducing such results requires recording the input meshes, hierarchy parameters, query sets, compiler settings, and hardware, and comparing the resulting paths and timings.

The custom navigation code also contains fixed-size buffers, experimental query variants, and threading experiments. Review their limits and behavior before adapting the implementation to other workloads.

## Credits and licensing

This repository is maintained by [Vahid Rahmani](https://github.com/rahmani-vahid).

It includes Recast/Detour code originally authored by Mikko Mononen, along with third-party components such as FastLZ, METIS, SDL-related binaries, and Improbable SDK headers. Preserve the copyright and license notices shipped with those components.

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
