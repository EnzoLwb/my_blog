---
title: FFMPEG 操作语音文件和图像视频文件
date: 2019-03-08 14:19:18
tags: 
    - FFMPEG   
category:
    - 服务端

---
> FFmpeg的名称来自MPEG视频编码标准，前面的“FF”代表“Fast Forward”，FFmpeg是一套可以用来记录、转换数字音频、视频，并能将其转化为流的开源计算机程序。可以轻易地实现多种视频格式之间的相互转换。FFmpeg的用户有Google，Facebook，Youtube，优酷，爱奇艺，土豆

1.  获取语音文件时长
#### ffmpeg -i {$audio_file_path/$url} 2>&1 | grep 'Duration' | cut -d ' ' -f 4 | sed s/,//
cut -d ',' -f 1

```
 Input #0, mp3, from 'spring.mp3':
  Metadata:
    genre           : Other
    encoder         : Lavf56.4.101
    album           : 麦兜菠萝油王子 电影原声大碟
    title           : 春田花花幼稚园校歌
    artist          : 何崇志
    disc            : 1
    track           : 2
  Duration: 00:00:49.06, start: 0.025056, bitrate: 488 kb/s
  
```
 
2. 将视频的第一帧图片作为视频封面
#### ffmpeg -i  {$video_file_path/$url}  -y -f mjpeg -ss 1 -t 1 -s 700x400  {$video_cover_path} 
- -y    覆盖输出文件，即如果输出文件已经存在的话，不经提示就覆盖掉了
- -f    强制输入或输出文件格式。通常会自动检测输入文件的格式，并从输出文件的文件扩展名中猜出格式，因此在大多数情况下不需要此选项。（不写这个参数还报错,但还是正确生成了图片）
- -ss   从指定时间开始转换
- -t    提取有限数量的帧 提起一个帧
- -s    输出的分辨率 700x400

3. 压缩视频文件  (压缩减小十倍都可以 不明显)
#### ffmpeg -i Wildlife.wmv -b:v 800k -s 960x540 aabb.mp4
- -b:v 输出文件的码率/比特率，一般500k左右即可，人眼看不到明显的闪烁，码率与视频大小是正比关系。
- -s 设置输出文件的分辨率

4. 压缩图片用intervention就ok了,不仅这些，还能做到视频合并，视频剪辑，视频变gif，视频提取音频，合并视频音频等等。