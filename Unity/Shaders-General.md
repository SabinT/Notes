# Docs on how properties are passed to shaders
https://docs.unity3d.com/Manual/SL-PropertiesInPrograms.html

## Look up texture size in shader
https://answers.unity.com/questions/678193/is-it-possible-to-access-the-dimensions-of-a-textu.html

## Loop unrolling errors in Mac (metal)
Looks like when indexing arrays with a variable that is not the loop variable (e.g., `i` or `j`) causes this error.

```glsl
// error: forced to unroll loop, but unrolling failed. (on metal)
for (int i = numballs - 3; i < numballs; i++)
{
    float h = float(i) / numballs;
    blobs[i].xyz = 0.5 * sin(6.2831*hash3(h*1.17) + hash3(h*13.7)*time);
}

// error: can't unroll loops marked with loop attribute (on metal)
UNITY_LOOP
for (int i = numballs - 3; i < numballs; i++)

{
    float h = float(i) / 8.0;
    blobs[i].xyz = 0.5 * sin(6.2831*hash3(h*1.17) + hash3(h*13.7)*time);
}

// error: forced to unroll loop, but unrolling failed.
for (int i = 0; i < 3; i++)
{
    float h = float(i) / 8.0;
// error: array reference cannot be used as an l-value; not natively addressable
    blobs[numballs - i].xyz = 0.5 * sin(6.2831*hash3(h*1.17) + hash3(h*13.7)*time);
}

// WORKS
for (int i = 0; i < 3; i++)
{
    float h = float(i) / 8.0;
    blobs[i].xyz = 0.5 * sin(6.2831*hash3(h*1.17) + hash3(h*13.7)*time);
}
```

## Gotcha - getting world position out of the matrix
https://answers.unity.com/questions/187628/shader-get-object-position-or-distinct-value-per-o.html

TLDR: the following might return (0,0,0) during batchings
```
float3 baseWorldPos = unity_ObjectToWorld._m03_m13_m23;
```