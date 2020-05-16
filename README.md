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

[#EXTM3U]
[#EXT-X-VERSION:3]
[#EXT-X-MEDIA-SEQUENCE:0]
[#EXT-X-ALLOW-CACHE:YES]
[#EXT-X-TARGETDURATION:12]
[#EXT-X-KEY:METHOD=AES-128,URI="https://127.0.0.1/video.key",IV=0x00000000000000000000000000000000]
[#EXTINF:12.000000,
https://127.0.0.1/enc-mp4-0.ts
[#EXTINF:9.000000,]
https://127.0.0.1/enc-mp4-1.ts
[#EXTINF:9.000000,]
https://127.0.0.1/enc-mp4-2.ts
[#EXTINF:0.033333,]
https://127.0.0.1/enc-mp4-3.ts
[#EXT-X-ENDLIST]

