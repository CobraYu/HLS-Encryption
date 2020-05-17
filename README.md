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


#### M3U8 Sample:( *IV only appears once in the header - the first IV used by OpenSSL* )




