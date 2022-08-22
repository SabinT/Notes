# Denoising
https://www.openimagedenoise.org/
https://github.com/DeclanRussell/IntelOIDenoiser/releases/tag/1.6

# Accumulation during play mode
The path tracer needs a camera render to happen to accumulate frames.
It may call `Update` etc multiple times during accumulation times.
You have to be careful if you want to automate saving frames manually without using the Recorder.

Nice thread here:
https://forum.unity.com/threads/path-tracing-wait-until-finished.1112650/

Snippet to capture 'N' samples within a single time step:
```csharp
    for (int i = 0; i < (rayTracingToggle.isOn ? samplesRaytracing : 0); i++)
    {
         activeCamera.Render();
    }
    ScreenCapture.CaptureScreenshot("data/image" + nextView + "-" + _number + ".jpg");
```

Another snippet worth looking into:
```csharp
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.HighDefinition;
 
public class WaitForConvergence: MonoBehaviour
{
    [Tooltip("Number of samples for convergence. Should be the same as the maximum samples in path tracing.")]
    public int Samples = 128;
 
    [Tooltip("The number of frames to capture. Set to -1 to continue capturing frames until the application exits.")]
    public int FramesToCapture = -1;
 
    bool _StopRecording = false;
    int _RecordedFrames = 0;
    Camera _camera;
 
    void Start()
    {
        _camera = GetComponent<Camera>();
 
        if (_camera == null)
            Debug.Log("You should attach this script to a camera game object");
    }
 
    void Update()
    {
        HDRenderPipeline renderPipeline = RenderPipelineManager.currentPipeline as HDRenderPipeline;
        if(renderPipeline != null && _StopRecording == false)
        {
            if(_RecordedFrames == 0)
                renderPipeline.BeginRecording(Samples, 1F, 0.25F, 0.75F);
 
            for (int i = 0; i < Samples - 1; i++)
            {
                renderPipeline.PrepareNewSubFrame();
                _camera?.Render();
            }
 
            ScreenCapture.CaptureScreenshot($"frame_{_RecordedFrames++}.png");
 
            if (_RecordedFrames == FramesToCapture)
            {
                _StopRecording = true;
                _RecordedFrames = 0;
                renderPipeline.EndRecording();
            }
        }
    }
}
```