---
title: Shader Build Pipeline
description: Learn about the Shader Build Pipeline in the Atom Renderer.
toc: true
weight: 200
---

  
The shader build pipeline is made of **Asset Builders**, coordinated by the **Asset Processor**, with the purpose of producing shader binaries that will be used by the runtime to render graphics.  
  
The most important asset builder in the shader build pipeline is the **ShaderAssetBuilder**. The ShaderAssetBuilder is the only required asset builder, it consumes files with extension `.shader` and produces shader binaries that are loaded in the **GPU** by the renderer. The remaining asset builders, **ShaderVariantListBuilder** and **ShaderVariantAssetBuilder**, exist only for performance purposes because they can generate optimized versions of the shader binaries known as **Shader Variants**.  
  
The following table summarizes the asset builders used by the shader build pipeline, along with the input source assets and output product assets.  

|Builder                   |Source Asset          |Intermediate Assets                             |Product Assets                       |
|---------                 |---------             |---------                                       |---------                            |
|ShaderAssetBuilder        |`*.shader`            |N/A                                             |`*.azshader` and `*.azshadervariant` |
|ShaderVariantListBuilder  |`*.shadervariantlist` |`*.hashedvariantlist` and `*.hashedvariantinfo` |N/A                                  |
|ShaderVariantAssetBuilder |`*.hashedvariantlist` |N/A                                             |`*.azshadervarianttree`              |
|ShaderVariantAssetBuilder |`*.hashedvariantinfo` |N/A                                             |`*.azshadervariant`                  |
  
Here is a brief overview of each asset builder  from the shader build pipeline:  
## ShaderAssetBuilder  
As mentioned above, this is the only required builder, the other builders can be disabled with the registry key `/O3DE/Atom/Shaders/BuildVariants` set to `false`. The ShaderAssetBuilder consumes Engine and Game files with paths like `Relative/Path/<ShaderName>.shader` and generates the following product assets in the `<Game>/Cache` folder:  
- `relative/path/<shadername>.azshader`. Notice that the relative path is all lower case, which is the **Asset Processor** standard. The runtime uses this type of assets through the `ShaderAsset` class. In turn, this product asset contains direct references to the **Root Shader Variant** assets with extension `.azshadervariant`, which contain the fully functional, yet unoptimized, version of the shader byte code. There will be one Root Shader Variant for each supported RHIs (dx12, vulkan and null) times the number of `Supervariants` specified inside the source `*.shader` file.
- `relative/path/<shadername>-<supervariantname>_<rhi>_0.azshadervariant`. There will be several of these Root Shader Variants. These files contain unoptimized byte code, which can be seen as "Uber Shaders".  
  
There are several applications that are sequentially invoked during shader compilation and all of these applications support command line arguments that users of O3DE can customize, starting with the files: `/o3de/Gems/Atom/Asset/Shader/Registry/atom_shaders.setreg`, and `/o3de/Gems/Atom/Asset/Shader/Config/shader_build_options.setreg`.  
  
For a deeper dive into functionality, features and customization options of the ShaderAssetBuilder see [The ShaderAssetBuilder User Guide](shader-asset-builder/).  
  
  
## ShaderVariantListBuilder
This is the first "optional" builder in the shader build pipeline. Its purpose is to parse **Shader Variant List** (`*.shadervariantlist`) files and produce "Intermediate Assets" with extensions  `*.hashedvariantlist` and `*.hashedvariantinfo`. In other words, this builder doesn't generate **Product Assets**, instead it generates **Intermediate Assets** that will be consumed by the **ShaderVariantAssetBuilder** to generate **optimized** shader binary assets known as **Shader Variant Assets**.  
  
A Shader Variant List file is an optional file, that represents a **customizable** list of Shader Variants that should be generated on behalf of a particular `*.shader` file. The O3DE Engine ships with a few `*.shadervariantlist` files, but the user can override them. There are path location rules and naming conventions that need to be followed if the user decides to override the Shader Variant List files that ship with O3DE.  
For example, the following shader file: `o3de/Gems/Atom/Feature/Common/Assets/Shaders/AuxGeom/AuxGeomWorld.shader` ships with this Shader Variant List: `o3de/Gems/Atom/Feature/Common/Assets/Shaders/AuxGeom/AuxGeomWorld.shadervariantlist`. Notice that the location and name of those two files are identical, except for their extensions, and this is the first rule to keep in mind... Shader Variant List files must be located in the same directory, and have the same name, as the Shader file when they ship with the Engine and/or Gems.  
If the user choses to override the Shader Variant List file mentioned above, they must create a new Shader Variant List file under the `<Game>/ShaderVariants/` folder, which in this particular example would resolve to:  `<Game>/ShaderVariants/Shaders/AuxGeom/AuxGeomWorld.shadervariantlist`, and this is the second rule to keep in mind... Shader Variant List files, customized by game projects, must be located under the `<Game>/ShaderVariants/` folder, concatenated with the "Scan Folder Relative" path of the original Shader file.  
  
For a deeper dive into functionality, features and customization options of the ShaderVariantListBuilder see The ShaderVariantListBuilder User Guide.  
  
## ShaderVariantAssetBuilder
This is the second "optional" builder in the shader build pipeline, and it consumes Intermediate Assets produced by the ShaderVariantListBuilder. This builder has two main purposes, it generates a ShaderVariantTreeAsset (`*.azshadervarianttree`) from a `*.hashedvariantlist` file, and produces the ShaderVariantAssets from each `*.hashedvariantinfo` file.  
The runtime uses the ShaderVariantTreeAsset as a means for fast lookup of Shader Variants, from the search result it loads the corresponding ShaderVariantAsset which will contain the most optimized version of the shader byte code.  
  
For a deeper dive into functionality, features and customization options of the ShaderVariantListBuilder see The ShaderVariantAssetBuilder User Guide.  
  



