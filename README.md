erver
preprocess 360° video through ffmpeg and bento4

# Configuration
## File directory
VR_team  
　└dataset　　    
　　├07.mp4　　
　　└CMP07.mp4　　
　└07  
　　└multibitrate  
　　　├CMP07_1M.mp4　　
　　　├CMP07_5M.mp4  
　　　└...  
　　└tile0  
　　　├tile0_CMP_07_1M.mp4  
　　　├f_tile0_CMP_07_1M.mp4  
　　　├audio.mp4  
　　　├f_audio.mp4  
　　　├...  
　　　└output  
　　　　└audio  
　　　　　└...  
　　　　└video  
　　　　　└...  
　　　　└stream.mpd  
　　└tile1  
　　　├tile1_CMP_07_1M.mp4  
　　　├f_tile1_CMP_07_1M.mp4    
　　　├...  
　　　└output  
　　　　└video  
　　　　　└...  
　　　　└stream.mpd      
　　└...  
　　└tile5  
　　　　└...  

notic:  
1. please put the orginal video file in `dataset`.  
2. only the fold `tile0` has audio part.  

## process.sh
process order:  
ERP2CMP --> multimedia.sh --> tile1.sh --> dash1.sh --> tile --> dash

##

