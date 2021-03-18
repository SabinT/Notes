## Useful Shader resources by platform

### Processing
https://processing.org/tutorials/pshader/

This page lists out the various uniforms/varyings available.

Some notes on 16 bit images in PS shaders:
https://forum.processing.org/two/discussion/9535/16-bit-per-channel-image-processing

### GLSL
Online syntax checker: http://shdr.bkcore.com

Often, the system that compiles GLSL returns little information about where the error is, the above website comes in handy to quickly fix syntax.

### Unity
TODO

### OpenFrameworks
TODO

### TouchDesigner
TODO

## Shader gotchas
* `fmod` in HLSL and `fmod` in GLSL return different results. Take care when porting code from one to the other.
