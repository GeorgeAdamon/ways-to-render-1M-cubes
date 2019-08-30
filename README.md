# Ways to Render 1 Million Cubes
All the ways to efficiently render 1 Million individually coloured cubes in Unity3d (I could think of).
**WIP: This repository will be filled with implementations over time**

## Concepts
### Positional Data Generation
### Cube Representation
### Rendering

## Implementations
### 1. Instanced Rendering

#### [1.1 Using MeshRenderers & MaterialPropertyBlock]()
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
* Performance is dependent on the total amount of triangles that need to be rendered (Instances x Triangles per Instance)
* No culling (occlusion or  frustrum) is performed automatically.
* Not supported by every platform.
* Sorting a million instances every frame, adds a computational overhead.

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
* Smallest possible memory footprint, since the grid does not need any buffers whatsoever, it's all written as a distance function, which is evaluated continuously at runtime.
##### Cons
* Arbitrary, cube-like repeating SDFs are **impossible** to implement, if they can't be described as solid operations.
* The cubes are incompatible-with and invisible-to any other Unity system (Real-time Lights, Lightmapping, Physics, Reflections etc)
* Occlusion from other Unity objects has to be handled inside the shader. (It's not automatic)
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

#### [4.2 Using the Visual Effects Graph (GPU)]()
##### Description
##### Pros
##### Cons

---
---

### 5. ECS
