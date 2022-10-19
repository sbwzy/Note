### 多段视频合并

```powershell
# mylist.txt 为存储需合并视频的文本文件
ffmpeg -f concat -i mylist.txt -c copy out1-6.mp4
```

```markdown
// mylist.txt
file 'out1.mp4'
file 'out2.mp4'
file 'out3.mp4'
file 'out4.mp4'
file 'out5.mp4'
file 'out6.mp4'
```

### 截取中间某段视频

```powershell
# 01:23:13为截取的开始时间戳，00:08:00代表截取8分钟长的视频
ffmpeg -ss 01:23:13 -i 2.mkv -to 00:08:00 -c copy out6.mp4
```

### 批量格式转换

```powershell
# 将文件夹下所有的.m4a文件转化为同名的.mp3文件
for %%a in (*.m4a) do ffmpeg -i "%%~na.m4a" "%%~na.mp3"
# 提取文件夹下所有的.flv文件的音频并输出同名的.mp3文件
for %%a in (*.flv) do ffmpeg -i "%%~na.flv" -vn "%%~na.mp3"
```