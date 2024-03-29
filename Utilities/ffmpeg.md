## Create from images in folder

For patterns like `image_000_0001.png, image_000_0002.png, img003.png`, etc:

> `ffmpeg -framerate 30 -i image_000_%04d.png -vcodec libx264 -crf 23 -vf format=yuv420p -movflags faststart -preset slow output.mp4`

For twitter, I also had to add `-vf format=yuv420p`, otherwise it wouldn't process the video:

> `ffmpeg -framerate 30 -i image_000_%04d.png -vcodec libx264 -crf 23 -preset slow -movflags faststart -vf format=yuv420p output.mp4`

See notes below for CRF.

## Optimize existing video for web
`ffmpeg -i input.mp4 -movflags faststart -acodec copy -vcodec copy output.mp4`

## Make gif
Without scaling:
> `ffmpeg -i image_000_%04d.png -vf "fps=50,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop 0 output.gif`

With scaling(width)
> `ffmpeg -i image_000_%04d.png -vf "fps=50,scale=1080:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop 0 output.gif`

Source:
https://superuser.com/questions/556029/how-do-i-convert-a-video-to-gif-using-ffmpeg-with-reasonable-quality

Note: `-loop 0` means infinite loop.

## Add watermark PNG
Add `watermark.png` to the bottom center of video:
`ffmpeg -i input.mp4 -i watermark.png -filter_complex "scale=200:100,overlay=(main_w-overlay_w)/2:(main_h-overlay_h)" -codec:a copy output.mp4`


## Get resolution of video

> `ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=s=x:p=0 video.mp4`

## Resize video

> `ffmpeg -i input.mp4 -vf scale=1920:1080 -preset slow -crf 18 output.mp4`

## Overlay PNG
> `ffmpeg -y -i video.mp4 -i overlay.png -filter_complex [0]overlay=x=0:y=0[out] -map [out] -map 0:a? output.mp4`

## Loop Video
Repeat 1 time:
`ffmpeg -stream_loop 1 -i input.mp4 -c copy output.mp4`

# Add audio to video
`ffmpeg -i input.mp4 -i input.mp3 -codec:a aac -c copy -map 0:v:0 -map 1:a:0 output.mp4`

If audio is shorter than video, use `-af apad -shortest` option, e.g:
`ffmpeg -i input.mp4 -i input.mp3 -codec:a aac -af apad -shortest output.mp4`

Note: AAC codec was needed to get the audio to play on an iPhone

## Crop Video
Example: crop to 1080:1080, auto center
>  `ffmpeg -i input.mp4 -filter:v "crop=1080:1080" output.mp4`

## Trim video

E.g., trim to keep from 6 seconds to 10 seconds.

> ` ffmpeg -ss 6 -i input.mp4 -to 10 -c copy -copyts output.mp4`

> BEWARE: do not change the order of parameters because that changes the meaning of the command! Also, there are other timestamp formats available. More info:
> https://trac.ffmpeg.org/wiki/Seeking#Cuttingsmallsections

## Loop with fade
30 secs vid with 1 sec fade:
```
ffmpeg -i video.mp4 -filter_complex "[0]split[body][pre];[pre]trim=duration=1,format=yuva420p,fade=d=1:alpha=1,setpts=PTS+(29/TB)[jt];[body]trim=1,setpts=PTS-STARTPTS[main];[main][jt]overlay" faded.mp4
```

## Reduce file size without resizing video

Lower CRF = higher quality / bigger file size. In my experiments (and also on the internet), 18 for x264 is quite high quality.

Use -preset slow/slower/veryslow for better quality (the slower, the better)

Warning: `-preset veryslow` is actually very slow

> `ffmpeg -i input.mp4 -vcodec libx264 -crf 20 -preset slow output.mp4`

WARNING: x265 is not widely supported

> `ffmpeg -i input.mp4 -vcodec libx265 -crf 23 -preset slow output.mp4`

## Trim based on start/end time
`-ss` seeks to given time.
`-to` is the duration from the seek time.
Time format is: `hh:mm:ss`.
Example: 2 seconds, starting at 1 second:
> `ffmpeg -ss 00:01:00 -to 00:02:00  -i input.mp4 -c copy output.mp4`

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
`ffmpeg -framerate 30 -i image_000_%04d.png -vcodec libx264 -crf 23 -vf "format=yuv420p, tmix=frames=3:weights='0.25 1 0.25'" -preset slow output.mp4`

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

## All files in folder (Mac/Linux)
```bash
for i in *.mov; do <command>; done
"$i" - original path
"${i%.*}.mp4" - original filename with extension changed to mp4
"${i%.*}-done.mp4" - similar to above with slightly modified filename
```

```bash
for i in *.mov; do ffmpeg -i "$i" -vcodec libx264 -crf 23 -vf format=yuv420p -preset slow "${i%.*}.mp4"; done
```

## All files in folder with Windows shell
Example: convert all `*.mp4` in folder to x264 with crf 20, save as `*_new.mp4`

```bat
FOR /F "tokens=*" %G IN ('dir /b *.mp4') DO ffmpeg -i "%G" -vcodec libx264 -crf 20 -preset slow -vf format=yuv420p "%~nG_new.mp4"
```