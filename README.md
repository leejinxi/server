server
preprocess 360° video through ffmpeg and bento4

# Configuration for MP4
## File directory
VR_team   
　├dataset  
　│　├8K.mp4  
　│　└CMP8K.mp4  
　└processed  
　  └CMP8K  
　　  ├multibitrate  
　　  │　├CMP8K_1M.mp4  
　　  │　├CMP8K_5M.mp4  
　　  │　└...  
　　  ├tile0  
　　  │　├tile0_CMP_CMP8K_1M.mp4  
　　  │　├f_tile0_CMP_CMP8K_1M.mp4  
  　　│　├audio.mp4  
　  　│　├f_audio.mp4  
  　　│　├...  
　　  │　└output  
　  　│　　├audio  
  　　│　　│　└...  
  　　│　　├video  
  　　│　　│　└...  
　  　│　　└stream.mpd  
　　  ├tile1  
  　　│　├tile1_CMP_CMP8K_1M.mp4  
  　　│　├f_tile1_CMP_CMP8K_1M.mp4    
  　　│　├...  
  　　│　└output  
  　　│　　├video  
  　　│　　│　└...  
  　　│　　└stream.mpd      
  　　├...  
  　　│  
  　　├...  
  　　└tile5  
　  　 　└...  

## process.sh
Process order:  
ERP2CMP --> multimedia.sh --> audio --> face.sh --> dash1.sh --> tile.sh --> dash2.sh    
Change the file location if need.  
Check the name of the input/output file in each step.  

## ERP2CMP:  
```
ffmpeg -i /dataset/8k.mp4 -vf v360=e:c3x2:cubic:w=8640:h=5760:outpad=0.01 -c:v libx264 -keyint_min 60 -g 60 -sc_threshold 0 -an /dataset/CMP8k.mp4
```
check parameters of v360 in: http://ffmpeg.org/doxygen/trunk/vf__v360_8c_source.html

## multibitrate.sh
src: root directory, in this part the src=/VR_team/processed   
name: the output of ERP2CMP file  
bitrate: unit `"M"`, please write in ascending order. If you want to change the unit to "k", you need to change all in files, often in the ffmpeg command lines.    

The output files will be located in `/VR_team/processed/CMP8k/multibitrate/`  

## face.sh
### cut CMP format video into 6 faces.
name: the output of ERP2CMP file  
src: root directory, in this part the src=/VR_team/processed/$name  
w: Number of widths divided  
h: Number of height divided
bitrate: same as above

The output files will be located in `$src/tile0/` `$src/tile1/` ...

## audio
### abstract the audio stream from orginal video.   
```
ffmpeg -i /dataset/8k.mp4 -vn -acodec copy -y /CMP8k/tile0/audio.mp4
```
Pay attention to the output location. Only the fold `tile0` has audio part.  

## dash1.sh
name: the output of ERP2CMP file  
bitrate: same as above  
src: root directory, in this part the src=/VR_team/processed/$name
```
mp4fragment --fragment-duration 2000 input.mp4 f_input.mp4 //2s
```
the fragmented file is located in /VR_team/processed/CMP_8K/tile`i`/

Increase or decrease the number of mp4dash input files according to the number of bit rate options, for example, bitrate=(1 5 10):  
```
mp4dash f_tile0_CMP8k_1M.mp4 f_tile0_CMP8k_5M.mp4 f_tile0_CMP8k_10M.mp4
```
In this file, you need to modify `"f_tile"$face"_"$name"_"${bitrate[j]}"M_mp4"`;`j`:the sequence number of array "bitrate". 

## tile.sh
### cut every face into `w*h` parts.  
name: the output of ERP2CMP file  
src: root directory, in this part the src=/VR_team/processed/$name  
w: Number of widths divided  
h: Number of height divided  
bitrate: same as above

## dash2.sh
same as `dash1.sh`

## NOTICE
1. please put the orginal video file in `dataset`.  
2. only the fold `face0` or `tile0` has audio part.  
3. For ffmpegGPU, you would like to change the `-c:v libx264` to `-c:v nvenc_h264`, but the video resolution should be < 4096.

# Configuration for Webm  
## playback Adaptive WebM using DASH  
Ref: http://wiki.webmproject.org/adaptive-streaming/instructions-to-playback-adaptive-webm-using-dash  

## WebM live streaming via DASH  
Ref: http://wiki.webmproject.org/adaptive-streaming/instructions-to-do-webm-live-streaming-via-dash

Server: nginx(nginx-rtmp-module)  
Push: OBS  
OBS streams the video to rtmp server `rtmp://222.20.77.111/ingest`, `key=obs`

### Generated the video stream (no audio here, 6 faces test):
`VP9_LIVE_PARAMS="-speed 6 -tile-columns 4 -frame-parallel 1 -threads 8 -static-thresh 0 -max-intra-rate 300 -deadline realtime -lag-in-frames 0 -error-resilient 1"`

```
ffmpeg -re -i rtmp://222.20.77.111:1935/ingest/obs \  
-map 0:v -an -pix_fmt yuv420p -c:v libvpx-vp9 -keyint_min 15 -g 15 ${VP9_LIVE_PARAMS} -vf "crop=w=in_w/3:h=in_h/2:x=0:y=0" -f webm_chunk -header "webmlive/obs_face1/obs_1.hdr" -chunk_start_index 1 webmlive/obs_face1/obs_1_%d.chk \  
... \
-map 0:v -an -pix_fmt yuv420p -c:v libvpx-vp9 -keyint_min 15 -g 15 ${VP9_LIVE_PARAMS} -vf "crop=w=in_w/3:h=in_h/2:x=0:y=0" -f webm_chunk -header "webmlive/obs_face6/obs_6.hdr" -chunk_start_index 1 webmlive/obs_face1/obs_6_%d.chk \  
```
### Create the DASH Manifest   
face1:
```
if [ -f "webmlive/obs_face6/obs_6.hdr" ];then  
    sleep 0.01s  
    ffmpeg -f webm_dash_manifest -live 1 -i webmlive/obs_face1/obs_1.hdr -c copy -map 0 -f webm_dash_manifest -live 1 -adaptation_sets "id=0,streams=0" -chunk_start_index 1 -chunk_duration_ms 2000 -time_shift_buffer_depth 1800 -minimum_update_period 1800 webmlive/obs_face1/index.mpd
fi
```
### Notice 
1. File name of the header should conform to the following format: <prefix>_<representation_id>.hdr  
2. File name of the chunks should conform to the following format: <prefix>_<representation_id>_%d.chk
3. Make sure that the "chunk_start_index" and the "chunk_duration_ms" parameters are the same as in the two steps.
4. Check the ffmpeg options: https://ffmpeg.org/ffmpeg-formats.html






