# server
preprocess 360° video through ffmpeg and bento4

# Configuration
## File directory
- VR_team   
丨- dataset    
  丨- 07.mp4  
  丨- CMP07.mp4  
丨- 07  
  丨- multibitrate  
    丨- CMP07_1M.mp4  
    丨- CMP07_5M.mp4
    丨- ...
  丨- tile0  
    丨- tile0_CMP_07_1M.mp4  
    丨- f_tile0_CMP_07_1M.mp4  
    丨- audio.mp4  
    丨- f_audio.mp4  
    丨- ...
    丨- output
      丨- audio
        丨- ...  
      丨- video  
        丨- ...  
      丨- stream.mpd  
  丨- tile1  
    丨- tile1_CMP_07_1M.mp4  
    丨- f_tile1_CMP_07_1M.mp4    
    丨- ...
    丨- output
      丨- video  
        丨- ...  
      丨- stream.mpd      
  丨- ...  
  丨- tile5  

notic:  
1. please put the orginal video file in `dataset`.  
2. only the fold `tile0` has audio part.  

## process.sh
process order:  
ERP2CMP --> multimedia.sh --> tile1.sh --> dash1.sh --> tile --> dash

##

