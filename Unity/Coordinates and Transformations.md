## Getting object scale in shader

```
// From: https://forum.unity.com/threads/can-i-get-the-scale-in-the-transform-of-the-object-i-attach-a-shader-to-if-so-how.418345/
float3 getWorldScale(float4x4 obj2world) {
  return float3(
      length(float3(obj2world[0].x, obj2world[1].x, obj2world[2].x)), // scale x axis
      length(float3(obj2world[0].y, obj2world[1].y, obj2world[2].y)), // scale y axis
      length(float3(obj2world[0].z, obj2world[1].z, obj2world[2].z))  // scale z axis
    );
}
```

## Getting object position/hash

```
float3 baseWorldPos = unity_ObjectToWorld._m03_m13_m23
```

Which can then be hashed with:

```
// from https://www.shadertoy.com/view/4djSRW
float hash13(float3 p3)
{
    p3  = frac(p3 * .1031);
    p3 += dot(p3, p3.zyx + 31.32);
    return frac((p3.x + p3.y) * p3.z);
}
```
