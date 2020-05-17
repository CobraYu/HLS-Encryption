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

( *IV only appears once in the header - the first IV used by OpenSSL* )

#### 5.关键文件(from "HTTP LIVE STREAMING DEVELOPMENT")
URI参数EXT - X的密钥标记标识的密钥文件。密钥文件中包含的密钥，解密随后的媒体在播放列表中的文件必须使用。
AES - 128加密方法使用16字节的密钥。*密钥文件的格式是简单地装在这16个字节的二进制格式的数组。*
5.1四，AES - 128
128位AES加密和解密时提供的相同的16字节的初始化向量（IV）。改变这四，增加密码的强度。
当使用AES - 128加密方法，实现应为IV使用的媒体文件的序列号，当媒体文件加密或解密。*big - endian的二进制表示的序列号应放置在一个16字节的缓冲区，并填充（左）零。*
如果加密方法是AES - 128，AES - 128 CBC encyption应适用于个别媒体文件。整个文件必须被加密。密码块链接，绝不能适用于整个媒体文件。*媒体文件的序列号，必须使用作为IV*，如5.1节所述。





