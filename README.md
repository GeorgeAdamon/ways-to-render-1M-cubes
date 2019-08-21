# Ways to Render 1 Million Cubes
All the ways to efficiently render 1 Million cubes in Unity3d (I could think of).
**WIP: This repository will be filled with implementations over time**

## ECS 


---
---
## Instanced Rendering

### [Using MeshRenderers & MaterialPropertyBlock]()
#### Description
#### Pros

---

### [Using DrawMeshInstanced]()
#### Description
#### Pros
#### Cons

---

### [Using DrawMeshInstancedIndirect]()
#### Description
_Using a single Mesh Instance, a single Material Instance and a custom Shader accepting arbitrary Buffers (like positions, rotations, colors etc.) and triggering the instanced drawing by calling the [Graphics.DrawMeshInstancedIndirect](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstancedIndirect.html) on every frame.
The shader is written to take care of the translation/scale/rotation of each instance._
#### Pros
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
* Arbitrary meshes can be used
* Sub-meshes can be instanced separately
* The transformation matrix for each element of the grid could be defined entirely on the shader
* Small memory footprint, since only 1 Mesh is allocated in memory
* Each instance is drawn in the correct order, allowing correct transparency effects.
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


