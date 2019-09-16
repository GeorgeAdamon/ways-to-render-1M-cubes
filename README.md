# Ways to Render 1 Million Cubes
All the ways to efficiently render 1 Million individually coloured cubes in Unity3d (I could think of).
**WIP: This repository will be filled with implementations over time**

## Index
* [Concepts -----------------------------------------------------------------------------------------------](#concepts)
  + [**Positional Data Generation**](#positional-data-generation)
  + [**Cube Representation**](#cube-representation)
  + [**Cube Grid Rendering**](#rendering)
* [Implementations ---------------------------------------------------------------------------------------](#implementations)
  + [**1. Instanced Rendering**](#1-instanced-rendering)
    - [_1.1 Using Plain Old Mesh Renderers (POMR) & MaterialPropertyBlock_](#11-using-plain-old-mesh-renderers-pomr--materialpropertyblock)
    - [_1.2 Using DrawMeshInstanced & MaterialPropertyBlock_](#12-using-drawmeshinstanced--materialpropertyblock)
    - [_1.3 Using DrawMeshInstancedIndirect & Custom Shaders_](#13-using-drawmeshinstancedindirect--custom-shaders)
    - [_1.4 Using DrawMeshInstancedIndirect with Custom GPU Z-Sorting_](#14-using-drawmeshinstancedindirect-with-custom-gpu-z-sorting)
    - [_1.5 Clustering Individual Instances into 3D Tiles_](#15-clustering-individual-instances-into-3D-tiles)
  + [**2. Procedural Meshing**](#2-procedural-meshing)
    - [_2.1 Generating the Cubes as a single Mesh_](#21-generating-the-cubes-as-a-single-mesh)
    - [_2.2 Generating the Cubes using a Geometry Shader_](#22-generating-the-cubes-using-a-geometry-shader)
    - [_2.3 "Stamping" the Cubes into a single Mesh_](#23-stamping-the-cubes-into-a-single-mesh)
  + [**3. Raymarching**](#3-raymarching)
    - [_3.1 Using a repeating Cube Signed Distance Field_](#31-using-a-repeating-cube-signed-distance-field)
    - [_3.2 Using a Volumetric RenderTexture_](#32-using-a-volumetric-rendertexture)
  + [**4. Particles**](#4-particles)
    - [_4.1 Using the Shuriken (CPU) Particle System_](#41-using-the-shuriken-cpu-particle-system)
    - [_4.2 Using the Visual Effects Graph (GPU)_](#42-using-the-visual-effects-graph-gpu)
  + [**5. ECS & Hybrid Renderer**](#5-ecs--Hybrid-Renderer)

---
---

## Concepts

### Positional Data Generation
#### Cached (Pre-generated)
#### On-The-Fly

### Cube Representation
#### Explicit (Mesh-Based)
#### Implicit (SDF)
#### Impostors (Single Quad)

### Cube Grid Rendering
#### Procedural Meshing
#### Instanced Rendering
#### Raymarched SDF

---
---

## Implementations

### 1. Instanced Rendering

#### [1.1 Using Plain Old Mesh Renderers (POMR) & MaterialPropertyBlock]()
##### Description
_Each cube is Instantiated as a separate GameObject with Transform, MeshFilter & MeshRenderer components attached to it. 
The MeshFilters of all the cubes point to a single (shared) Mesh asset. 
The MeshRenderers of all the cubes point to a single (shared) Material asset, which supports [GPU Instancing](https://docs.unity3d.com/Manual/GPUInstancing.html). 
Each cube is individually coloured via a [MaterialPropertyBlock](https://docs.unity3d.com/ScriptReference/MaterialPropertyBlock.SetColor.html)_

##### Pros
* Arbitrary, cube-like meshes can be easily used.
* Compatible with every platform by default.
* Easier to debug, since objects actually exist in the Scene and can be individually inspected.
* The cubes can be interacted with & exhibit scripted behaviour.
* Unity handles culling (occlusion and frustrum), batching & Z-sorting efficiently and natively.
* Works with real-time Shadows, Lightmapping, Light/Reflection Probes, Physics. 

##### Cons
* Adds Editor overhead, since the Scene file will now contain 1 million individual objects.
* Increases the project size, since each instance's description is embedded into the file.
* Memory-hungry since each **visual** instance is also a separate GameObject (Transform + MeshFilter + MeshRenderer).
* Performance is dependent on the total amount of triangles that need to be rendered (Instances x Triangles per Instance).
* Updating the MaterialPropertyBlock requires serial iteration through all of the instances.

---

#### [1.2 Using DrawMeshInstanced & MaterialPropertyBlock]()
##### Description
_Using a single Mesh Instance, a single Material Instance and a Matrix4x4 array containing the transformation of each instance.
The instanced drawing is trigerred by calling the [Graphics.DrawMeshInstanced](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstanced.html) on every frame._

##### Pros
* Arbitrary, cube-like meshes can be easily used.
* Smaller memory footprint, since only 1 GameObject is needed in the Scene.
* The shader does not need to be customized, therefore built-in or 3rd party shaders can be used out of the box.
* Works with real-time Shadows & Light/Reflection Probes. 

##### Cons
* Limited to 1023 instances per call.
* No Z-Sorting is performed automatically, which becomes really problematic with transparent materials.
* No culling (occlusion or  frustrum) is performed automatically.
* Not supported by every platform ???
* Performance is dependent on the total amount of triangles that need to be rendered (Instances x Triangles per Instance)
* Updating the MaterialPropertyBlock requires serial iteration through all of the instances.

---

#### [1.3 Using DrawMeshInstancedIndirect & Custom Shaders]()
##### Description
_Using a single Mesh Instance, a single Material Instance and a custom Shader accepting arbitrary Buffers (like positions, rotations, colors etc.) and triggering the instanced drawing by calling the [Graphics.DrawMeshInstancedIndirect](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstancedIndirect.html) on every frame.
The shader is written to take care of the translation/scale/rotation of each instance._

##### Pros
* 1 Draw Call for the whole grid.
* Arbitrary, cube-like meshes can be easily used.
* Smaller memory footprint, since only 1 GameObject is needed in the Scene.
* The transformation matrix for each element of the grid could be composed directly on the shader.

##### Cons
* No Z-Sorting is performed automatically, which becomes really problematic with transparent materials.
* No culling (occlusion or  frustrum) is performed automatically.
* Not supported by every platform.
* Performance is dependent on the total amount of triangles that need to be rendered (Instances x Triangles per Instance)

---

#### [1.4 Using DrawMeshInstancedIndirect with Custom GPU Z-Sorting]()
##### Description
_Using a single Mesh Instance, a single Material Instance, a custom Shader accepting arbitrary Buffers (like positions, rotations, colors etc.) and a Compute Shader that sorts the indices of the instances in parallel(Bitonic Merge Sort), based on their distance to the Camera. The instanced drawing is trigerred by calling the [Graphics.DrawMeshInstancedIndirect](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstancedIndirect.html) on every frame, after the execution of the sorting.
The shader is written to take care of the translation/scale/rotation of each instance, and draw the instances in the correct order, with the ones further away from the camera drawn first._

##### Pros
* 1 Draw Call for the whole grid.
* Arbitrary, cube-like meshes can be easily used.
* Smaller memory footprint, since only 1 GameObject is needed in the Scene.
* The transformation matrix for each element of the grid could be composed directly on the shader.
* Each instance is drawn in the correct Z order, allowing correct transparency effects.

##### Cons
* Bitonic GPU Sorting works correctly only if **2*N = 2<sup>k</sup>**, with N = Number of elements to sort.
* Performance is dependent on the total amount of triangles that need to be rendered (Instances x Triangles per Instance)
* No culling (occlusion or  frustrum) is performed automatically.
* Not supported by every platform.
* Sorting a million instances every frame, adds a computational overhead.

#### [1.5 Clustering Individual Instances into 3D Tiles]()
##### Recipe
_Instead of using 1 cube per instance, generate a Mesh that groups together multiple cubes, and use that as each instance's mesh. The more cubes each instance represents, the less instances need to be drawn. Conversely, the more cubes each instance represents, the larger the memory footprint. Edge case is when the mesh contains **all** of the cubes, therefore is rendered as one instance, storing **all** of its vertices in memory. See [2.1](#21-generating-the-cubes-as-a-single-mesh)_
##### Pros
##### Cons
* Assigning **Per-Cube** properties, like Color, becomes trickier, since the number of colors will be larger than the number of instances being drawn. This has to be handled in the shader.

---
---

### 2. Procedural Meshing

#### [2.1 Generating the Cubes as a single Mesh]()
##### Description
_A single mesh containing the Vertices, Indices, Normals and Colors of all the cubes is generated procedurally, and passed to the MeshFilter of a GameObject. The buffers (vertex,index,normal,color etc) of the Mesh are generated either in the CPU using the Job System or in the GPU using a ComputeShader._
##### Pros
* 1 Draw Call for the whole grid.
* Compatible with every platform by default.
* Unity handles Z-sorting efficiently and natively.
* Works with real-time Shadows, Lightmapping, Light/Reflection Probes.
##### Cons
* Arbitrary, cube-like meshes are difficult to implement, since the geometry has to be described mathematically.
* The upload of a huge monolithic mesh to the GPU memory, introduces a big performance spike.
* No culling (occlusion or  frustrum) is performed, since the cubes belong to one object.

---

#### [2.2 Generating the Cubes using a Geometry Shader]()
##### Description
##### Pros
##### Cons
* Surface shading is not supported in geometry shaders, so all the light calculations have to be implemented **from scratch**.

---

#### [2.3 "Stamping" the Cubes into a single Mesh]()
##### Description
##### Pros
##### Cons

---
---

### 3. Raymarching

#### [3.1 Using a repeating Cube Signed Distance Field]()
##### Description
##### Pros
* 1 Draw Call for the whole grid.
* Compatible with every platform by default.
* Arbitrary, cube-like repeating SDFs are **very** easy to implement, if they can be described as solid operations.
* Performance is **independent** of the total amount of instances that need to be rendered. Rendering 1 Cube is the same as 1 million cubes.
* **Smallest possible memory footprint**, since the grid does not need any buffers whatsoever, it's all written as a distance function, which is evaluated continuously at runtime.
##### Cons
* Custom cube shapes are **impossible** to implement, if they can't be described as solid operations.
* The cubes are incompatible-with and invisible-to any other Unity system (Real-time Lights, Lightmapping, Physics, Reflections etc)
* Occlusion from other Unity objects has to be handled inside the shader. (It's not automatic)
* Shadows, Diffuse Lighting and Reflections have to be implemented as raymarching functions inside the shader.
* Writing to the Camera's Normal and Depth Buffer (for letting post-effects "see" the grid) has to be implemented from scratch.

---

#### [3.2 Using a Volumetric RenderTexture]()
##### Description
##### Pros
##### Cons

---
---

### 4. Particles

#### [4.1 Using the Shuriken (CPU) Particle System]()
##### Description
##### Pros
##### Cons

---

#### [4.2 Using the Visual Effects Graph (GPU)]()
##### Recipe
_1. Generate a signed floating-point (R32G32B32) Texture, which encoded the positions of the cubes' centers as colors. (XYZ => RGB)._</br>
_2. Spawn particles once, and use **Set Position From Map** at their birth using the Texture._</br>
_3. Use a Mesh Output node, and choose the appropriate Cube Mesh_.
##### Pros
* Minimal code required, just for updating the Texture's values if the grid parameters change.
##### Cons
* The Visual Effects Graph doesn't work with the Legacy Unity rendering pipeline. Requires either URP/LWRP or HDRP.

---
---

### 5. ECS & Hybrid Renderer
