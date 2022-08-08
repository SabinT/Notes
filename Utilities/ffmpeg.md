## Create from images in folder

For patterns like `img001.png, img002.png, img003.png`, etc:

> `ffmpeg -framerate 30 -i img%03d.png -vcodec libx264 -crf 23 -vf format=yuv420p -preset slow output.mp4`

For twitter, I also had to add `-vf format=yuv420p`, otherwise it wouldn't process the video:

> `ffmpeg -framerate 30 -i img%03d.png -vcodec libx264 -crf 23 -preset slow -vf format=yuv420p output.mp4`

See notes below for CRF.

## Get resolution of video

> `ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=s=x:p=0 video.mp4`

## Resize video

> `ffmpeg -i input.mp4 -vf scale=1920:1080 -preset slow -crf 18 output.mp4`

## Trim video

E.g., trim to keep from 6 seconds to 10 seconds.

> ` ffmpeg -ss 6 -i input.mp4 -to 10 -c copy -copyts output.mp4`

> BEWARE: do not change the order of parameters because that changes the meaning of the command! Also, there are other timestamp formats available. More info:
> https://trac.ffmpeg.org/wiki/Seeking#Cuttingsmallsections

## Reduce file size without resizing video

Lower CRF = higher quality / bigger file size. In my experiments (and also on the internet), 18 for x264 is quite high quality.

Use -preset slow/slower/veryslow for better quality (the slower, the better)

Warning: `-preset veryslow` is actually very slow

> `ffmpeg -i input.mp4 -vcodec libx264 -crf 20 -preset slow output.mp4`

WARNING: x265 is not widely supported

> `ffmpeg -i input.mp4 -vcodec libx265 -crf 23 -preset slow output.mp4`

## CRF values for x264

> Use 18 for high quality

The range of the quantizer scale is 0-51: where 0 is lossless, 23 is default, and 51 is worst possible. A lower value is a higher quality and a subjectively sane range is 18-28. Consider 18 to be visually lossless or nearly so: it should look the same or nearly the same as the input but it isn't technically lossless.

https://superuser.com/questions/677576/what-is-crf-used-for-in-ffmpeg

## CRF values for x265

> 28 for x265 compares to 23 for x264

> Heuristic: +2 compared to x264 CRF

Source:
https://trac.ffmpeg.org/wiki/Encode/H.265

## FFmpeg filters

Format:

```
ffmpeg ... rest of command .. -vf "filter1, filter2, ..."
```

E.g., the following has two filters for format and tmix:
`-vf "format=yuv420p, tmix=frames=3:weights='0.25 1 0.25'"`

## Temporal convolution
Sample Command:
`ffmpeg -framerate 30 -i img%03d.png -vcodec libx264 -crf 23 -vf "format=yuv420p, tmix=frames=3:weights='0.25 1 0.25'" -preset slow output.mp4`

This allows mixing multiple frames for stuff like averaging.
Weights are centered around 'current frame'.
Funky looking past and future trails: `tmix=frames=3:weights='-1 3 -1'`
Extreme motion blur: `tmix=frames=7:weights='1 1 1 1 1 1 1'`
Small amount of motion blur (weighted): `tmix=frames=3:weights='0.25 1 0.25'`

## Sources

https://ottverse.com/change-resolution-resize-scale-video-using-ffmpeg/

## x264 vs x265

h265 (or HEVC) is supposed to succeed h264, but according to Mozilla, browser support is limited, and it looks like adoption is encumbered with patents
https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Video_codecs#hevc_h.265

As of 2021, h264 appears better in terms of compatibility.

I read that if you want to play very high resolution videos in realtime, HEVC is better (e.g., for 8K videos), because of better hardware decoding performance in GPUs.

## All files in folder with Windows shell
Example: convert all `*.mp4` in folder to x264 with crf 20, save as `*_new.mp4`

```bat
FOR /F "tokens=*" %G IN ('dir /b *.mp4') DO ffmpeg -i "%G" -vcodec libx264 -crf 20 -preset slow -vf format=yuv420p "%~nG_new.mp4"
```