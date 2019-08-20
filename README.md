# Ways to Render 1 Million Cubes
All the ways to efficiently render 1 Million cubes in Unity3d (I could think of).
**WIP: This repository will be filled with implementations over time**

## ECS 

<br/>

## Instanced Rendering

### Using MeshRenderers & MaterialPropertyBlock
#### Description

### Using DrawMeshInstanced
#### Description

### Using DrawMeshInstancedIndirect
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
* No Z-Sorting is performed automatically. Problematic with transparent materials.
* No culling is performed automatically.

### Using DrawMeshInstancedIndirect with Custom GPU Z-Sorting
<br/>

## Procedural Meshing

### Generating the Cubes as a single Mesh
#### Description
### Generating the Cubes using a Geometry Shader
#### Description


<br/>

## Raymarching

### Using a repeating Cube Signed Distance Field
#### Description
### Using a Volumetric RenderTexture
#### Description

