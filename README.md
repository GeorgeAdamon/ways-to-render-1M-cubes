# Ways to Render 1 Million Cubes
All the ways to efficiently render 1 Million individually coloured cubes in Unity3d (I could think of).
**WIP: This repository will be filled with implementations over time**

## ECS 


---
---
## Basic Instanced Rendering

### [Using MeshRenderers & MaterialPropertyBlock]()
#### Description
_Each cube is Instantiated as a separate GameObject with Transform, MeshFilter & MeshRenderer components attached to it. 
The MeshFilters of all the cubes point to a single (shared) Mesh asset. 
The MeshRenderers of all the cubes point to a single (shared) Material asset, which supports [GPU Instancing](https://docs.unity3d.com/Manual/GPUInstancing.html). 
Each cube is individually coloured via a [MaterialPropertyBlock](https://docs.unity3d.com/ScriptReference/MaterialPropertyBlock.SetColor.html)_
#### Pros
* Compatible with every platform by default.
* Unity handles culling and Z-sorting efficiently and natively.
* Somewhat easier to debug, since objects actually exist in the Scene and can be individually inspected.
* The cubes can be interacted with & exhibit scripted behaviour.
* Works with real-time Shadows, Lightmapping, Light/Reflection Probes, Physics. 
#### Cons
* Memory-hungry since each **visual** instance is also a separate GameObject (Transform + MeshFilter + MeshRenderer).
* Adds Editor overhead, since the Scene file will now contain 1 million individual objects.
* Increases the project size, since each instance's description is embedded into the file.
* Updating the MaterialPropertyBlock requires serial iteration through all of the instances.
* Performance is dependent on the total amount of triangles that need to be rendered (Instances x Triangles per Instance).

---

### [Using DrawMeshInstanced]()
#### Description
_Using a single Mesh Instance, a single Material Instance and a Matrix4x4 array containing the transformation of each instance.
The instanced drawing is trigerred by calling the [Graphics.DrawMeshInstanced](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstanced.html) on every frame._
#### Pros
* Arbitrary meshes can be used
* Sub-meshes can be instanced separately
* The shader does not need to be customized, therefore built-in or 3rd party shaders can be used out of the box.
* Small memory footprint, since only 1 Mesh is allocated in memory
#### Cons
* Limited to 1023 instances per call.
* Performance is dependent on the total amount of triangles that need to be rendered (Instances x Triangles per Instance)
* No Z-Sorting is performed automatically, which becomes really problematic with transparent materials.
* No culling is performed automatically.

---

### [Using DrawMeshInstancedIndirect]()
#### Description
_Using a single Mesh Instance, a single Material Instance and a custom Shader accepting arbitrary Buffers (like positions, rotations, colors etc.) and triggering the instanced drawing by calling the [Graphics.DrawMeshInstancedIndirect](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstancedIndirect.html) on every frame.
The shader is written to take care of the translation/scale/rotation of each instance._
#### Pros
* 1 Draw Call for the whole grid
* Arbitrary meshes can be used
* Sub-meshes can be instanced separately
* The transformation matrix for each element of the grid could be defined entirely on the shader
* Small memory footprint, since only 1 Mesh is allocated in memory
#### Cons
* Performance is dependent on the total amount of triangles that need to be rendered (Instances x Triangles per Instance)
* No Z-Sorting is performed automatically, which becomes really problematic with transparent materials.
* No culling is performed automatically.
* Not supported by every platform.

---

### [Using DrawMeshInstancedIndirect with Custom GPU Z-Sorting]()
#### Description
_Using a single Mesh Instance, a single Material Instance, a custom Shader accepting arbitrary Buffers (like positions, rotations, colors etc.) and a Compute Shader that sorts the indices of the instances in parallel(Bitonic Merge Sort), based on their distance to the Camera. The instanced drawing is trigerred by calling the [Graphics.DrawMeshInstancedIndirect](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstancedIndirect.html) on every frame, after the execution of the sorting.
The shader is written to take care of the translation/scale/rotation of each instance, and draw the instances in the correct order, with the ones further away from the camera drawn first._
#### Pros
* 1 Draw Call for the whole grid
* Arbitrary meshes can be used
* Sub-meshes can be instanced separately
* The transformation matrix for each element of the grid could be defined entirely on the shader
* Small memory footprint, since only 1 Mesh is allocated in memory
* Each instance is drawn in the correct Z order, allowing correct transparency effects.
#### Cons
* Performance is dependent on the total amount of triangles that need to be rendered (Instances x Triangles per Instance)
* No culling is performed automatically.
* Not supported by every platform.
* Sorting a million instances every frame, adds a computational overhead.

---
---

## Procedural Meshing

### [Generating the Cubes as a single Mesh]()
#### Description
#### Pros
#### Cons

---

### [Generating the Cubes using a Geometry Shader]()
#### Description
#### Pros
#### Cons

---
---

## Raymarching

### [Using a repeating Cube Signed Distance Field]()
#### Description
#### Pros
#### Cons

---

### [Using a Volumetric RenderTexture]()
#### Description
#### Pros
#### Cons


