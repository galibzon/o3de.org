---
title: Shader Asset Builder User Guide
description: Learn about the ShaderAssetBuilder in the shader build pipeline.
toc: true
weight: 200
---

  
## ShaderAssetBuilder  
The ShaderAssetBuilder consumes `*.shader` files, which will refer as **Shaders** or **Shader Source Assets** for the rest of this document. The main purposes of this builder is to generate the shader binaries, as product assets, that wll be used by the runtime to render graphics. We'll refer to shader binary product assets as **Shader Assets** for the rest of this document. Typically the Material System and some Passes make direct reference to the Shader Assets.  
Shaders are located in folders that are visible to the **Asset Processor**, typically distributed as part of Gems and Game Projects, just like any other source asset.  
Shaders do not contain shader code (AZSL, HLSL, etc), instead, these are JSON files that contain metadata, which serve as build recipes and command line arguments for the tools involved in compiling AZSL files into shader binaries. For details into the JSON format of `*.shader` files you can read [Shader File Specification](./shader-file-spec).  
Looking at the ShaderAssetBuilder as an opaque system of input and outputs, the input is a Shader Source Asset that is located in a Gem or Game Project relative folder that looks like: `Relative/Path/<ShaderName>.shader` and the output will be product assets in the `<Game>/Cache/<platform>/relative/path/` folder:  
- `relative/path/<shadername>.azshader`. Notice that the relative path is all lower case, which is the **Asset Processor** standard. The runtime uses this type of assets through the `ShaderAsset` class. In turn, this product asset contains direct references to the **Root Shader Variant** assets with extension `.azshadervariant`, which contain the fully functional, yet unoptimized, version of the shader byte code. There will be one Root Shader Variant for each supported RHIs (dx12, vulkan and null) times the number of `Supervariants` specified inside the source `*.shader` file.
- `relative/path/<shadername>-<supervariantname>_<rhi>_0.azshadervariant`. There will be several of these Root Shader Variants. These files contain unoptimized, yet fully functional, shader byte code, which can be seen as "Uber Shaders".  
  
This is the the sequence of tools and invocations that the ShaderAssetBuilder uses to compile shaders for each platform supported by the runtime:  

### Windows
\<ShaderName\>.shader  -> 

There are several applications that are sequentially invoked during shader compilation and all of these applications support command line arguments that users of O3DE can customize, starting with the files: `/o3de/Gems/Atom/Asset/Shader/Registry/atom_shaders.setreg`, and `/o3de/Gems/Atom/Asset/Shader/Config/shader_build_options.setreg`.  
  
For a deeper dive into functionality, features and customization options of the ShaderAssetBuilder see The ShaderAssetBuilder User Guide.  
  
| Topic                        | Description |
|--------------------------------------|---------|
| [The Amazon Shading Language (AZSL)](azsl/) | AZSL is an open source extension of HLSL. With AZSL you can author shaders in one language and build them for multiple target platforms. | 
| [The Amazon Shading Language Compiler (AZSLc)](azslc/) | AZSLc transpiles AZSL code into HLSL code. | 
| [Shader Build Pipeline](shader-build-pipeline) | The shader build pipeline is a subsystem of the **Asset Pipeline** that builds shaders based on compilation attributes you define. When **Asset Processor** discovers `.azsl` source assets in scan directories, the shader build pipeline uses AZSLc, DirectX Shader Compiler (DXC), and SPIRV-Cross to compile the shader product assets for the specified target platforms. |
| [Shader File Specification](/docs/atom-guide/look-dev/shaders/shader-file-spec/) | A JSON reference for `.shader` files. |


In O3DE, shaders are source assets with extension `.shader`. The **Asset Processor** scans for `*.shader` files and proceeds to compile those shaders with the **ShaderVariantAssetBuilder**, which will produce shader binaries for each supported platform.  
  
A `.shader` file is a json file with metadata that describes compilation recipes to 

 regular assets in O3DE,  and as such they are compiled  are written in the **Amazon Shading Language (AZSL)**, coupled with specialized configuration files that add various metadata.

Shaders are made up of several files:  

- **`*.azsl`**: The main AZSL source file that contains the shader program.
- **`*.shader`**: References the .azsl file and attaches metadata to configure the shader for compiling. 
- **`*.azsli`**: (Optional) Contains reusable AZSL code, which is intended to be included in .azsl files. 
- **`*.srgi`**: (Optional) Contains AZSL code, which combines partial SRGs.
- **`.shadervariantlist`**: (Optional) Describes what shader variants must be compiled for a given .shader file. 
  
## AZSL Files

**AZSL** is a variation of HLSL, with a few extensions that make it easier to author and maintain shaders. Most programming rules that apply to AZSL are the same for HLSL. You can refer to the [AZSL and Compiler](azsl/) section and the [Microsoft DirectX HLSL](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-reference) documentation for more details.  

There are a few key extensions of AZSL, which affect the shader build pipeline:
- **Shader Resource Groups (SRGs)**: A container for application visible variables, which are defined in code with the class `ShaderResourceGroup`. SRGs can be defined in `.azsl` files as *partial SRGs*, which define only a portion of an SRG. You can combine the partial SRGs in a single `.srgi` file. Learn more about [SRGs](/docs/atom-guide/dev-guide/shaders/azsl/#shader-resource-groups) in the AZSL section.

- **Shader variant options**: Application visible variables, which the developer can choose to compile as static constants or as regular global variables. The compiled shader code results in **shader variants**, or variations of the shader code, which minimize branching in runtime. You can specify the variants you want to pre-build in a `.shadervariantlist` file. Learn more about [shader variant options](/docs/atom-guide/dev-guide/shaders/azsl/shader-variant-options).

### AZSL file (`.azsl`)

The `.azsl` file is the main source file that contains AZSL code and shader programs. Atom currently supports vertex, pixel (or fragment), compute, and raytracing shaders. The `.azsl` file might also include other files with reusable AZSL code, like `.azsli` and `.srgi` files.


### Shader Asset Files (`*.shader`)

The `.shader` files are written in JSON. They reference the main AZSL source file (`*.azsl`) and add a variety of metadata for configuring AZSLc and indicating how the render pipeline should use this shader. 

### AZSL Include file (`.azsli`)

The `.azsli` files contain AZSL code that is meant to be reused and shared among multiple `.azsl` files. The `.azsli` extension is simply a naming convention to indicate that the file contains reusable AZSL code and should be included by `.azsl` files; otherwise, `.azsli` files are virtually identical to the `.azsl` files. 

### SRG Include file (`.srgi`)

The `.srgi` files are specialized AZSL files that are used to merge multiple partial SRG definitions into a single SRG asset. This allows multiple Gems to contribute their own resources to a common SRG, like the SceneSrg or ViewSrg. 

## Shaders in the Asset Pipeline

The **Asset Processer** has several builders that work together to process shader files and produce all of the assets that are needed in runtime. These shader assets and shader variant assets are most often used by the pass system or by the material system. 

The shader build pipeline consists of the following processes: 
1. The **Shader Asset Builder** converts the `.shader` file into a `.azshader` file, known as a *shader asset*. It also produces a shader variant asset (`*.azshadervariant`) for the root shader variant, which is the main variant of the shader and is the default bytecode that's used for rendering.
2. The **Amazon Shading Language Compiler (AZSLc)** and platform compilers compile the `.azsl` file by transpiling AZSL to HLSL (Shader Model 6.3).
3. The target platform's shader compiler compiles the HLSL code.

The Shader System uses the following shader compilers for supported platforms: 
- **D3D12**: DirectX shader compiler with DXT emission.
- **Vulkan**: DirectX shader compiler with SpirV emission.
- **Metal**: DirectX shader compiler and [SPIRV-Cross](https://github.com/KhronosGroup/SPIRV-Cross) to generate metalSL emission. 
