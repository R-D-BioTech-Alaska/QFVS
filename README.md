# QFVS — Quantum Field Vision

**A rendering engine built around the state of an entire world, not just the next frame.**

I'm developing QFVS because a world doesn't stop existing when the camera turns away from it. Terrain, buildings, materials, objects, and changes made miles from the player still belong to the same world. I want to represent all of that without making the renderer carry a dense, constantly resident copy of everything.

Modern engines already have good culling, LOD, streaming, and virtualized geometry. I'm not pretending those problems have never been addressed. What I'm testing is whether we can change the *underlying representation of the world* enough to reduce the work before it even reaches the GPU.

QFVS is an independent C++20/Vulkan rendering project from **R&D BioTech Alaska**. The goal is a complete, practical renderer for large, persistent, heavily authored worlds: correct geometry and materials, moving and multiple cameras, real lighting, reliable offscreen state, and measurable performance on ordinary hardware.

## The idea

A conventional scene is often managed as a very large collection of objects, buffers, textures, and spatial structures. That works, but large worlds contain an enormous amount of structure: repeated data, related regions, spatial hierarchies, common source pages, and relatively small changes to otherwise stable content.

QFVS is built around using that structure instead of throwing it away.

The world maintains its own versioned state. Active views request what they need. The renderer selects an appropriate representation, reconstructs geometry or material data when useful, and submits actual work to the GPU. An offscreen edit must remain part of the world even if nothing is currently drawing that region.

This does **not** mean rendering every square meter every frame. It means keeping the *world* coherent while only paying the rasterization cost for views that actually need pixels.

### What I'm building into it

- **Persistent world state.** Authored regions, objects, materials, edits, and versions with source identities that can be independently checked and replayed.
- **Sparse and structured representations.** Hierarchical spatial data, shared pages, content-addressed storage, and quantized tensor train (QTT) methods where the data genuinely supports them.
- **Adaptive reconstruction.** Multiple levels of detail, selective source reconstruction, bounded residency, and incremental updates rather than rebuilding a whole world after a local change.
- **Multi-camera streaming.** One coherent world with view-dependent demand, culling, and residency decisions that account for more than one camera.
- **Native GPU rendering.** Vulkan 1.3, vertex/index buffers, indirect draws, shader and material bindings, textures, lighting, and physical GPU framebuffer validation.
- **Verifiable correctness.** Independent CPU references, source hashes, exact replay, corruption rejection, and pixel/depth/object-ID comparisons. Where approximation is used, its error needs to be measured.

The intended data path looks like this:

```text
Authored terrain, meshes, materials, objects, edits
                        |
             Versioned world state
          (regions / epochs / identity)
                        |
         Sparse / QTT / shared source data
                        |
          Camera demand + retained state
                        |
         LOD / culling / residency / streaming
                        |
       Geometry and material reconstruction
                        |
              Vulkan draw execution
                        |
        Framebuffer readback and validation
```

These are parts of one engine design, not a claim that the entire pipeline has already passed physical GPU acceptance.

## Why the quantum part?

QFVS is influenced by my work on [Qubit State Algebra (QSA)](https://github.com/R-D-BioTech-Alaska/QSA): preserving mathematical structure, keeping separable information separate, and doing work on the part of a state that actually changed.

That does **not** make QFVS a quantum computer. It does not require a QPU, and QTT compression is not proof of a quantum speedup. This is research into using structured mathematical representations inside a conventional CPU/GPU graphics engine.

Some data compresses or reconstructs exceptionally well. Other data doesn't. When a structured method costs more or produces unacceptable visual error, the correct result is to use a conventional representation. I care about the whole cost, not just a nice compression ratio in isolation.

## Development status

**As of November 9, 2026:** QFVS is in active native development. The public repository is currently the project home, not a downloadable engine release.

| Area | Current state |
| --- | --- |
| C++20 native engine foundation | Implemented and tested locally |
| Source-authenticated world state, replay, sparse residency and streaming | Bounded native and CPU validation completed |
| Authored geometry, material data, camera/lighting interfaces and independent CPU references | Implemented in tested development paths; production integration remains incomplete |
| Vulkan 1.3 pipeline, descriptor, command and resource-lifetime integration | Native builds and controlled preflight tests completed |
| Original-engine physical Vulkan queue submission, completed hardware fences and pixel readback | **Not yet independently accepted** |
| Full production multi-region scenes and matched real-GPU performance benchmarks | **Not yet established** |
| Public source distribution, binaries and build instructions | **Not yet released** |

The latest accepted original-engine work is the local **E239/R91 native** development line. Passing those native tests is not the same thing as producing a verified hardware-rendered frame. Compiled shaders, simulated GPU calls, or successful command recording don't count as an actual GPU result.

Separate QSA-native texture and page-atlas experiments have produced useful CPU-side and shader-validation results. They are not being passed off as integrated renderer improvements. Some material-fidelity tests failed their acceptance thresholds; those representations are not ready to replace the original material data.

There is **no verified end-to-end FPS advantage, VRAM reduction, or universal exponential scaling claim** for the complete QFVS renderer at this stage.

## How I intend to measure it

A fast result is only useful when it renders the same thing correctly.

QFVS results need to account for:

1. **Visual correctness:** the same source scene, camera, geometry, object identities, material behavior, depth, and lighting, compared against an independent reference.
2. **Total resource cost:** CPU time, resident and transferred bytes, allocation overhead, GPU memory, bandwidth, and reconstruction cost. Saving bytes in one place while spending more elsewhere is not a win.
3. **Actual GPU execution:** real driver submission, completed hardware fences, readback images, and measured frame times—not a mock or a compiled pipeline.
4. **World behavior:** distant state, offscreen edits, cold loading, camera movement, multiple viewpoints, and source changes while the world is running.
5. **Fair baselines:** optimized conventional rendering, streaming, and compression methods at matched fidelity. Comparing against a deliberately inefficient dense implementation tells me very little.

That includes comparisons with established large-world techniques such as Unreal Engine's World Partition, Nanite, instancing, and virtual texturing where appropriate. If a QFVS method loses to a conventional method, that result belongs in the record too.

## What comes next

The immediate priority is a verified physical Vulkan frame from the original renderer, with real GPU submission, fence completion, and independently compared color, depth, and object-ID output.

From there, the work moves through production-authored cameras and materials, proper PBR lighting, multi-region rendering, GPU residency and streaming under pressure, and matched hardware benchmarks. Structured QSA/QTT methods only become default paths when they win on both fidelity and total cost.

Diosa is one of the intended real-world proving grounds: a large, continuous game world rather than a collection of convenient benchmark scenes. QFVS, QSA, and Diosa remain separate projects. Their private assets and source do not become part of this repository simply because they inform the research.

## Source and builds

There are no public build or installation steps yet because the native engine source has **not** been released in this repository. I won't put commands here that don't work.

When the source is ready for publication, this section will include the exact dependencies, toolchain, reproducible build steps, sample scenes, and validation procedure. Until then, this repository documents the project, its direction, and the limits of the results being claimed.

## License and authorship

The files **published in this repository** are provided under the [MIT License](LICENSE). That does not automatically license unpublished research code, Diosa assets, or third-party material.

**Brenton Carter**  
R&D BioTech Alaska  
[www.RDBioTech.org](https://www.rdbiotech.org/) · [GitHub](https://github.com/R-D-BioTech-Alaska)

I'm not trying to win a benchmark by making most of the world disappear from the test. I want to build a renderer that can keep the world intact, show the right pixels, and prove exactly where the savings come from.
