# Some interesting types of noise
## Perlin Noise
Good introduction in below link. Also has link to some example implementations.
https://thebookofshaders.com/11/

Great place to test out combinations with other math functions:
https://graphtoy.com/

## GPU Gems article on noise implementation
https://developer.nvidia.com/gpugems/gpugems2/part-iii-high-quality-rendering/chapter-26-implementing-improved-perlin-noise

GPU Gems source code:
https://github.com/QianMo/GPU-Gems-Book-Source-Code/tree/master/GPU-Gems-2-CD-Content/High-Quality_Rendering/Ch_26_Implementing_Improved_Perlin_Noise

## GPU Implementation example
https://github.com/ashima/webgl-noise/tree/master/src

## Cellular Noise
Also often called Voronoi noise/cellular noise.
Good discussion here:
https://thebookofshaders.com/12/

## Platform independent GPU implementation for GLSL
lumina.sourceforge.net/Tutorials/Noise.html

Short example:
```
https://greentec.github.io/shadertoy-fire-shader-en/
uniform float a;
uniform float b;
uniform float c;

float rand(vec2 co) {
  return fract(sin(dot(co.xy, vec2(a, b))) * c);
}

void main() {
    float x = rand(gl_FragCoord.xy);
    gl_FragColor = vec4(x, x, x, 1.0);
}
```

Another example:
```glsl
// From https://thebookofshaders.com/11/

// 2D Random
float random (in vec2 st) {
    return fract(sin(dot(st.xy,
                         vec2(12.9898,78.233)))
                 * 43758.5453123);
}

// 2D Noise based on Morgan McGuire @morgan3d
// https://www.shadertoy.com/view/4dS3Wd
float noise (in vec2 st) {
    vec2 i = floor(st);
    vec2 f = fract(st);

    // Four corners in 2D of a tile
    float a = random(i);
    float b = random(i + vec2(1.0, 0.0));
    float c = random(i + vec2(0.0, 1.0));
    float d = random(i + vec2(1.0, 1.0));

    // Smooth Interpolation

    // Cubic Hermine Curve.  Same as SmoothStep()
    vec2 u = f*f*(3.0-2.0*f);
    // u = smoothstep(0.,1.,f);

    // Mix 4 coorners percentages
    return mix(a, b, u.x) +
            (c - a)* u.y * (1.0 - u.x) +
            (d - b) * u.x * u.y;
}
```
