# Cosine Palettes
Credits: Inigo Quilez
https://iquilezles.org/articles/palettes/

HLSL implementation:
```
float3 palette(float t, float3 a, float3 b, float3 c, float3 d )
{
    return a + b*cos(6.28318*(c*t+d) );
}
```


## Exmple: Blue
```
    float3 col = palette(
        t,
        float3(0.2,0.5,0.8),  // bias
        float3(0.1,0.3,0.4),  // amplitude multilier
        float3(1.0,1.1,1.2),  // frequency
        float3(0.0,0.0,0.0) // phase
    );
```