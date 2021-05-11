## Color Space Gotchas
RenderTextures and imported textures might be in different color spaces (linear vs gamma). When copying pixels from an imported texture onto a RenderTexture (e.g., via a Compute Shader), the image may look unnaturally brighter or darker.

You can try one of the following to achieve parity:
1. Uncheck "sRGB" in texture import settings if feasible.
2. Convert color space in the compute shader. E.g., the following snippet

```hlsl
        // Blit with gamma correction
        float4 col = Image[id.xy];
        float3 corrected = pow(col.rgb, 0.454545);
        Result[id.xy] = float4(corrected, col.a);
```
