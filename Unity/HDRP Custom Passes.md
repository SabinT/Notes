## Stuff that you need to know

- Things are rendered in camera-space. So double check to make sure if the "world position" is camera-relative or absolute.
  https://docs.unity3d.com/Packages/com.unity.render-pipelines.high-definition@11.0/manual/Camera-Relative-Rendering.html

- The semantics of a Custom pass, stuff available to shaders, etc:
  https://docs.unity3d.com/Packages/com.unity.render-pipelines.high-definition@11.0/manual/Custom-Pass-Creating.html#Custom-Renderers-Pass-shader

- Custom buffers can be sampled in shaders, or in the Shader Graph using small snippets.

```hlsl
#if SHADERGRAPH_PREVIEW == 0
#include "Packages/com.unity.render-pipelines.high-definition/Runtime/RenderPipeline/RenderPass/CustomPass/CustomPassCommon.hlsl"
#endif

void SampleCustomColor_float(float2 UV, out float4 Color)
{
#if SHADERGRAPH_PREVIEW == 1
    Color = float4(0, 0, 0, 1);
#else
    Color = SampleCustomColor(UV);
#endif
}

void SampleCustomDepth_float(float2 UV, out float Depth)
{
// Note: the syntax for the below #ifdef has changed
#if SHADERGRAPH_PREVIEW == 1
    Depth = 0;
#else
    Depth = LinearEyeDepth(SampleCustomDepth(UV), _ZBufferParams);
#endif
}

}
```

## Raymarching in HDRP Custom Pass

// TODO

## Debugging

The Frame Debugger allows visualizing the results after every step of rendering.
https://docs.unity3d.com/Manual/FrameDebugger.html
