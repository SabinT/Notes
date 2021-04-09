## General
Incomplete but useful book/playground: https://thebookofshaders.com/
Signed Distance fields (2D): https://www.iquilezles.org/www/articles/distfunctions2d/distfunctions2d.htm

Inspiration videos:
https://www.youtube.com/watch?v=8--5LwHRhjk
https://www.youtube.com/watch?v=-pdSjBPH3zM


## Useful Shader resources by platform

### Processing
https://processing.org/tutorials/pshader/

This page lists out the various uniforms/varyings available.

Some notes on 16 bit images in PS shaders:
https://forum.processing.org/two/discussion/9535/16-bit-per-channel-image-processing

### GLSL
Online syntax checker: http://shdr.bkcore.com

Often, the system that compiles GLSL returns little information about where the error is, the above website comes in handy to quickly fix syntax.

## Shader gotchas
* `fmod` in HLSL and `fmod` in GLSL return different results. Take care when porting code from one to the other.
* Metal/MacOS doesn't seem to support 64-bit floats in shaders.
