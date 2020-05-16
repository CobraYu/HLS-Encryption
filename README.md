# HLS-Encryption

## FFmpeg + OpenSSL

## FFmpeg
#### 1.Check the codec and container format of the input file:

ffprobe -i $input

#### 2.Segment the input file and output the m3u8 list file and corresponding mpegts files:

ffmpeg -i $input -vcodec libx264 -acodec libvo_aacenc -b:v 128k -flags -global_header -map 0:0 -map 0:1 -f segment -segment_time 10 -segment_list_size 0 -segment_list list.m3u8 -segment_format mpegts %d.ts

ffmpeg -i $input -c copy -b:v 128k -b:a 48k -flags -global_header -map 0:0 -map 0:1 -f segment -segment_time 10 -segment_list_size 0 -segment_list list.m3u8 -segment_format mpegts %d.ts

ffmpeg -i $input -c copy -flags -global_header -map 0:0 -map 0:1 -f segment -segment_time 10 -segment_list_size 0 -segment_list list.m3u8 -segment_format mpegts %d.ts

#### 3.Encrypt the mpegts files by OpenSSL:

[#]!/bin/bash

keyFile=”video.key”
openssl rand 16 > $keyFile
encryptionKey=`cat $keyFile | hexdump -e ’16/1 “%02x”‘`

splitFilePrefix=”stream”
encryptedSplitFilePrefix=”enc/${splitFilePrefix}”

numberOfTsFiles=`ls ${splitFilePrefix}*.ts | wc -l`

for (( i=0; i<$numberOfTsFiles; i++ ))
do
initializationVector=`printf ‘%032x’ $i`
openssl aes-128-cbc -e -in ${splitFilePrefix}$i.ts -out ${encryptedSplitFilePrefix}$i.ts -nosalt -iv $initializationVector -K $encryptionKey

done

#### Keypoint: *Initialization Vector must be the sequence numbers of mpegts files.*


#### M3U8 Sample:(IV only appears once in the header - the first IV used by OpenSSL)


### Reference:
1. PROTECTED HLS USING FFMPEG AND OPENSSL - https://dryize.wordpress.com/2014/04/02/protected-hls-using-ffmpeg-and-openssl/
2. 在Nginx上為你的網站加入Https - https://medium.com/@zneuray/%E5%9C%A8nginx%E4%B8%8A%E7%82%BA%E4%BD%A0%E7%9A%84%E7%B6%B2%E7%AB%99%E5%8A%A0%E5%85%A5https-32af0223283a 

