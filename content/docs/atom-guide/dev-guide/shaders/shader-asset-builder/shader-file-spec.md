---
title: "Shader File Specification"
description: "A file specification for Shader files (`*.shader`) in the Graphics Renderer."
toc: false
weight: 1000
---

Shader files (`*.shader`) are written in JSON format. The ultimate source of truth into what are the property elements in these files, is the [AZ::RPI::ShaderSourceData](https://github.com/o3de/o3de/blob/0fa6eea55514ef710fa4a0d40ea31ad43b181de9/Gems/Atom/RPI/Code/Include/Atom/RPI.Edit/Shader/ShaderSourceData.h) class.  
  
The Shader files describe how an AZSL file should be compiled to produce the desired shader byte code, along with reflection data so the runtime understands the layout of the shader constants and resources.  
Here is a detailed description of the properties:

- **Source**: A file path to the AZSL source file (`*.azsl`). The path can be relative to this shader file or relative to the asset root.
  
- **DrawList**: The name of the draw list where DrawItems that are using this shader should be queued for rendering. This name must match the draw list name in one or more passes (`*.pass` file) to be rendered.
  
- **DepthStencilState**: The depth stencil state for the output merger.
  - **depth**
    - **enable**: `0` or `1`. Default `1` (Depth testing is enabled).
    - **writeMask**: Default `All`.
      - `Zero`: Do not update the depth buffer.
      - `All`: Update the depth buffer.
    - **compareFunc**: Default `Less`. The test function to compare the depth value of the incoming fragment against the depth buffer.
      - `Never`
      - `Less`
      - `Equal`
      - `LessEqual`
      - `Greater`
      - `NotEqual`
      - `GreaterEqual`
      - `Always`
  - **stencil**: 
    - **enable**: `0` or `1`. Default `0` (Stencil testing is disabled).
    - **readMask**: An 8bits number in the range 0x00 to 0xFF. 
    - **writeMask**: An 8bits number in the range 0x00 to 0xFF.
    - **frontFace**: Front face stencil op.
      - **failOp**: Default `Keep`.
        - `Keep`
        - `Zero`
        - `Replace`
        - `IncrementSaturate`
        - `DecrementSaturate`
        - `Invert`
        - `Increment`
        - `Decrement`
      - **depthFailOp**: Default `Keep`.
        - ... Same properties as **failOp**.
      - **passOp**: Default `Keep`.
        - ... Same properties as **failOp**.
      - **func**: Default `Always`.
        - `Never`
        - `Less`
        - `Equal`
        - `LessEqual`
        - `Greater`
        - `NotEqual`
        - `GreaterEqual`
        - `Always`
    - **backFace**: Back face stencil op.
      - ... Same properties as **frontFace**.
  
- **RasterState**: The render state configuration for the rasterizer. 
  - **depthBias**: Default `0`.
  - **depthBiasClamp**: Default `0.0f`.
  - **depthBiasSlopeScale**: Default `0.0f`.
  - **fillMode**: Default `Solid`.
    - `Solid`
    - `Wireframe`
  - **cullMode**: Default `Back`.
    - `None`
    - `Front`
    - `Back`
  - **multisampleEnable**: Default `0`.
  - **depthClipEnable**: Default `1`.
  - **conservativeRasterEnable**: Default `0`.
  - **forcedSampleCount**: Default `0`.
  
- **BlendState**:
  
- **GlobalTargetBlendState**:
  
- 
- **ProgramSettings**: A container for shader program settings.
  - **EntryPoints**: The list of shader entry points to build. If the EntryPoints setting is omitted, the builders will look for valid functions starting or ending with VS, PS, or CS, corresponding to vertex, fragment, and compute shaders.
    - **Name**: The name of the shader entry point function, which was defined in the AZSL source file (`.azsl`). 
    - **Type**: The type of shader function. The supported types are “Vertex”, “Fragment”, and “Compute”. 
  


{{< note >}}
This is a high level breakdown of the available elements in the shader asset file. The most reliable way to know the full specification is to inspect `ShaderSourceData::Reflect()` in `Gems\Atom\Code\Source\RPI.Edit\Shader\ShaderSourceData.cpp`. 

For the various render states listed above, like DepthStencilState and RasterState, see `Gems\Atom\RHI\Code\Source\RHI.Reflect\RenderStates.cpp`.
{{< /note >}}


The following is an example of a shader file, `MinimalPBR_ForwardPass.shader`. 
```json
{
    "Source" : "./MinimalPBR_ForwardPass.azsl",

    "DepthStencilState" :
    {
        "Depth" :
        {
            "Enable" : true,
            "CompareFunc" : "GreaterEqual"
        },
        "Stencil" :
        {
            "Enable" : true,
            "ReadMask" : "0x00",
            "WriteMask" : "0xFF",
            "FrontFace" :
            {
                "Func" : "Always",
                "DepthFailOp" : "Keep",
                "FailOp" : "Keep",
                "PassOp" : "Replace"
            }
        }
    },

    "CompilerHints" : { 
        "DxcDisableOptimizations" : false
    },

    "ProgramSettings":
    {
      "EntryPoints":
      [
        {
          "name": "MinimalPBR_MainPassVS",
          "type": "Vertex"
        },
        {
          "name": "MinimalPBR_MainPassPS",
          "type": "Fragment"
        }
      ]
    },

    "DrawList" : "forward"
}

```
