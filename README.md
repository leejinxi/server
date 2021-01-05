server
preprocess 360° video through ffmpeg and bento4

# Configuration for MP4
## File directory
VR_team   
　├dataset  
　│　├8K.mp4  
　│　└CMP8K.mp4  
　├server//this  
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
in this file, you need to modify `"f_tile"$face"_"$name"_"${bitrate[j]}"M_mp4"`,`j`:the sequence number of array "bitrate". 

## tile.sh
### cut every face into wxh parts.  
name: the output of ERP2CMP file  
src: root directory, in this part the src=/VR_team/processed/$name  
w: Number of widths divided  
h: Number of height divided  
bitrate: same as above

## dash2.sh
same as `dash1.sh`

# NOTICE
1. please put the orginal video file in `dataset`.  
2. only the fold `face0` or `tile0` has audio part.  
3. For ffmpegGPU, you would like to change the `-c:v libx264` to `-c:v nvenc_h264`, but the video resolution should be < 4096.


# Configuration for Webm

