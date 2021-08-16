Bug记录
1. java.lang.RuntimeException: Failure delivering result ResultInfo{who=null, request=1000, result=-1, data=Intent { (has extras) }}
 to activity {com.example.previewdemo/com.example.previewdemo.test.MainActivity}: java.lang.SecurityException: Media projections
 require a foreground service of type ServiceInfo.FOREGROUND_SERVICE_TYPE_MEDIA_PROJECTION
在使用华为手机进行录屏时出现该错误.
解决方法: 将targetSdkVersion改成28及以下

2.E/MPEG4Writer: do not support out of order frames (timestamp: 4708360 < last: 4709468 for Audio track
      Dumping Audio track's last 10 frames timestamp and frame type
音轨时间戳混乱: 时间戳是在编码时通过当前时间计算出来的.经测试,该时间没有出现混乱的情况.而出现混乱的是另一个时间戳-编码后的时间戳


//测试所有参数
//写文件解耦
//音频,视频参数提供

/*关于采集器,编码器,混合器开始与结束的顺序分析*/
开始顺序: 1.采集器 -> 2.编码器 -> 3.混合器 (或2-> 3 -> 1)
混合器一定要比编码器后初始化,因为混合器只有在添加音轨或视轨之后才能进行初始化,即addTrack()后才能调start()
采集器与编码器的顺序并无要求,它俩相互不影响.此处将采集器放在前面只是因为更符合人的思维

结束顺序: 2.编码器 -> 1.采集器 -> 3.混合器 (或2 -> 3 -> 1)
为什么编码器要比采集器先停止?
    因为停止编码器前,它要给最后一条编码的数据设置结束标志.
    如果此时采集器先停止,那编码器会因为获取不到数据无法给最后一条编码数据设置结束标志
为什么编码器要比混合器先停止?
    因为混合器也要等带有结束标志的编码好的数据,如果混合器先停止,那合成的视频就不携带结束标志了.
至于混合器和采集器的顺序并无要求.
那我们会有一个问题: 结束标志真的必要吗?如果不需要,那结束顺序就可以是任意的了.



=======ffmpeg 命令=======
查看每帧信息: ffprobe -print_format xml -show_frames previewSdk_camera.h264
