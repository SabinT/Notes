## Get resolution of video

`ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=s=x:p=0 video.mp4`

## Resize video

`ffmpeg -i input.mp4 -vf scale=1920:1080 -preset slow -crf 18 output.mp4`

## Reduce file size without resizing video

> Lower CRF = higher quality / bigger file size

> Use -preset slow/slower/veryslow for better quality (the slower, the better)

> Warning: `-preset veryslow` is actually very slow

`ffmpeg -i input.mp4 -vcodec libx264 -crf 20 -preset slow output.mp4`

`ffmpeg -i input.mp4 -vcodec libx265 -crf 23 -preset slow output.mp4`

## CRF values for x264

> Use 18 for high quality

The range of the quantizer scale is 0-51: where 0 is lossless, 23 is default, and 51 is worst possible. A lower value is a higher quality and a subjectively sane range is 18-28. Consider 18 to be visually lossless or nearly so: it should look the same or nearly the same as the input but it isn't technically lossless.

https://superuser.com/questions/677576/what-is-crf-used-for-in-ffmpeg

## CRF values for x265

> 28 for x265 compares to 23 for x264

> Heuristic: +2 compared to x264 CRF

Source:
https://trac.ffmpeg.org/wiki/Encode/H.265

## Sources

https://ottverse.com/change-resolution-resize-scale-video-using-ffmpeg/
