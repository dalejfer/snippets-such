# ffmpeg command snippets and guide

## webm

simple encoding  

```
ffmpeg -i input.mp4 -c:v libvpx -crf 10 -b:v 1M -c:a libvorbis output.webm
```

more configurations  

```
ffmpeg -i <input> -codec:v libvpx -quality good -cpu-used 0 -b:v 500k -qmin 10 -qmax 42 -maxrate 500k -bufsize 1000k -threads 4 -codec:a libvorbis -b:a 128k out.webm
```

```
ffmpeg -i <in> -ss 00:00:00 -t 30 -c:v libvpx -quality good -cpu-used 0 -b:v 300k -crf 9 -qmin 0 -qmax 50 -maxrate 300k -minrate 100k -bufsize 800k -threads 2 -c:a libvorbis -b:a 128k output.webm
```
  

## mp4 - mkv

simple encoding  

```
ffmpeg -i <input> -c:v libx264 -preset slow -crf 22 -c:a aac -b:a 160k -pix_fmt yuv420p <output>
```

### crf for h264 and h265

H.264: crf 17 - 28 (real 0-51)
H.265: crf 


-pix_fmt yuv420p


## some video filters

### scale

If we'd like to keep the aspect ratio, we need to specify only one component, either width or height, and set the other component to -1. For example, this command line:  

```
ffmpeg -i input.mp4 -vf scale=-1:320 output_320.mp4
```

### flip

#### horizontal flip (mirror)

```
-vf hflip
```

#### vertical flip (upside-down)

```
-vf vflip
```

### crop

```
ffmpeg -i in.mp4 -filter:v "crop=out_w:out_h:x:y" out.mp4
```

Where the options are as follows:  

`out_w` is the width of the output rectangle  
`out_h` is the height of the output rectangle  
`x` and `y` specify the top left corner of the output rectangle  
  

### fade -in / out (video)

para hacer fade-in comenzando en el segundo 13 con transicion de 1 segundo
y fade-out comenzando en el segundo 90 con transicion de 3 segundos.
recordar que se toma el tiempo del video de entrada (input) ...aparentemente.  

```
-vf "fade=t=in:st=13:d=1, fade=t=out:st=90:d=3"
```

### rotate (transpose)

```
ffmpeg -i in.mov -vf "transpose=1" out.mov
```

0 = 90CounterCLockwise and Vertical Flip (default)
1 = 90Clockwise
2 = 90CounterClockwise
3 = 90Clockwise and Vertical Flip

Use `-vf "transpose=2,transpose=2"` for 180 degrees.


### blur effect

 ```
ffmpeg -i derpdog.mp4 -filter_complex \
 "[0:v]crop=200:200:60:30,boxblur=10[fg]; \
  [0:v][fg]overlay=60:30[v]" \
-map "[v]" -map 0:a -c:v libx264 -c:a copy -movflags +faststart derpdogblur.mp4
```

[source, superuser](https://superuser.com/questions/901099/ffmpeg-apply-blur-over-face)


### subtitles

ffmpeg -ss 5:00.00 -copyts -i video.avi -ss 5:00.00 -vf subtitles=subtitles.srt out.avi

https://trac.ffmpeg.org/wiki/HowToBurnSubtitlesIntoVideo


## join several videos into one

### concat demuxer

first crate a file `mylist.txt`:

```txt
# mylist.txt
# this is a comment
file '/path/to/file1'
file '/path/to/file2'
file '/path/to/file3'
```

then execute:  

```
ffmpeg -f concat -safe 0 -i mylist.txt -c copy output
```

### concat video filter

**3 videos with audio**

```powershell
ffmpeg -i opening.mkv -i episode.mkv -i ending.mkv \
-filter_complex "[0:v] [0:a] [1:v] [1:a] [2:v] [2:a] \
concat=n=3:v=1:a=1 [v] [a]" \
-map "[v]" -map "[a]" output.mp4
```

**2 videos with no audio**

```powershell
ffmpeg -i .\1.mp4 -i .\2.mp4 -filter_complex "[0:v] [1:v] concat=n=2:v=1 [v]" -map "[v]" -an output.mp4
```

source: https://stackoverflow.com/questions/7333232/how-to-concatenate-two-mp4-files-using-ffmpeg

## slideshow with images

Concat demuxer: You can use the concat demuxer to manually order images and to provide a specific duration for each image.  

First, make a text file with the appropriate info:  

```txt
# mylist.txt
file '/path/to/dog.png'
duration 5
file '/path/to/cat.png'
duration 1
file '/path/to/rat.png'
duration 3
file '/path/to/tapeworm.png'
duration 2
file '/path/to/tapeworm.png'
```
  
(Due to a quirk, the last image has to be specified twice - the 2nd time without any duration directive)  

Then run the ffmpeg command:  

```
ffmpeg -f concat -i input.txt -vsync vfr -pix_fmt yuv420p output.mp4
```

## audio encoding

### aac codec

```
-c:a aac -b:a 160k
```

#### para versiones mas viejas (2.8)

```
-c:a aac -static -2 -b:a 160k
```
  
### mp3 codec

```
-c:a libmp3lame -b:a 192k
```


## video con 1 imagen y 1 archivo de audio

```
ffmpeg -loop 1 -i <img-input> -i <audio-inut> -shortest -c:v libx264 -c:a copy <output>
```

```
ffmpeg -loop 1 -i <img-input> -i <audio-input> -shortest -c:v libvpx -quality good -cpu-used 0 -b:v 500k -qmin 10 -qmax 42 -maxrate 500k -bufsize 1000k -threads 4 -c:a libvorbis -b:a 128k output.webm
```


## slow down / speed up video

double up speed:  

```
-filter:v "setpts=0.5*PTS"
```

slow down (use value greater than 1):  

```
-filter:v "setpts=2.0*PTS"
```

## slow down audio

```
-af "atempo=0.5"
```


## GIF

first create the color pallete  

```
ffmpeg -v warning -i <input> -vf "fps=15,scale=320:-1:flags=lanczos,palettegen" -y palette.png
```

then create the gif:  

```
ffmpeg -v warning -i <input> -i palette.png -lavfi "fps=15,scale=320:-1:flags=lanczos [x]; [x][1:v] paletteuse" -y <output>
```


## cut


```
-ss <start> -to <stop>
```

or

```
-ss <start> -t <duration>`
```

## metadata (simple)

```
-metadata title="Title"
```

## 2pass encoding

For two-pass, you need to run ffmpeg twice, with almost the same settings, except for:  

In pass 1 and 2, use the -pass 1 and -pass 2 options, respectively.  
In pass 1, output to a null file descriptor, not an actual file. (This will generate a logfile that ffmpeg needs for the second pass.)  
In pass 1, you need to specify an output format (with -f) that matches the output format you will use in pass 2.  
In pass 1, specify the audio codec used in pass 2; in many cases, -an in pass 1 will not work.   
  
For example:  

```
ffmpeg -y -i input -c:v libx264 -b:v 2600k -pass 1 -c:a aac -b:a 128k -f mp4 /dev/null && \
ffmpeg -i input -c:v libx264 -b:v 2600k -pass 2 -c:a aac -b:a 128k output.mp4
```
  

> Note: Windows users should use NUL instead of /dev/null and ^ instead of \.  
  

## get img out of a specific time in video

```
ffmpeg -i input_movie.mp4 -ss 00:00:05 -f image2 -vframes 1 imagename.png
```

`-i`              > The input video file  
`-ss  00:00:05`   > Start at Second 5 of movie  
`-f image2`       > Force image output  
`-vframes 1`      > Set the number of video frames to record  


## ffmpeg 2 videos side by side

```
ffmpeg -i desk.webm -i webcam.webm -filter_complex "[1:v][0:v]scale2ref=main_w:ih[sec][pri];[sec]setsar=1,drawbox=c=black:t=fill[sec];[pri][sec]hstack[canvas]; [canvas][1:v]overlay=main_w-overlay_w" out.mp4
```
  

## BLOB

```
ffmpeg -protocol_whitelist file,http,https,tcp,tls,crypto -i "" -bsf:a aac_adtstoasc -c copy out.mp4
```