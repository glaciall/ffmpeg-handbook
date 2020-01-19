[toc]

## ffmpeg入门指南
ffmpeg是音视频处理方面的瑞士军刀，目前市面上几乎所有的音视频工具，都与ffmpeg脱不了关系。我们在音视频处理或开发工作时，有ffmpeg这个工具，能够帮我们解决大部分的问题，或是在我们的应用里集成ffmpeg，将能够拥有非常强大的音视频处理能力，而ffmpeg或是说音视频方面的水非常的深，需要学习的知识点太多太多，光ffmpeg命令行的文档，就有3万行左右，还不是完全版的，还有音视频方面，也有大量的知识点需要去了解，完全不下于ffmpeg本身。总的来说，学习曲线相当陡峭。

本文档旨在于说明ffmpeg的一般过程、基础的架构，让大家能够掌握ffmpeg的学习与使用的方法。

### 基础概念
1. 编码codec：是对于原始的音视频的数字表示方式，通常是无压缩形式的，或是无结构的。
2. 封装格式format：可能会对原始的音视频进行压缩（有损或无损），并且有很明确的包结构，用于描述音视频的元信息。

### 框架图
<img src="img/framework.png" />

ffmpeg一般完成音频或视频的转码或是处理的事情，它可以同时支持多个输入，也可以支持多个输出，在输入与输出之中，完成一些你想要做的事情，比如转码、加视频滤镜、音视频分离、整合等等的事情。下面说明一下ffmpeg在上面框图上每一步可能会做的事情。

1. 读取输入：ffmpeg一般会通过输入的形式，去猜解输入数据源的封装形式，数据编码等。普通文件，可能会通过文件后缀名进行猜测封装形式，也肯定会对数据内容直接进行猜测，但这是对于普通的比较知名的封装形式而言，但是对于PCM音频类的就不奏效了，音频通常是无封装形式的，是裸数据，这个时候，我们通常需要声明format，比如-f s16le。在声明输入参数时，一定要想办法让ffmpeg知道，将要读取的数据源，它是什么封装形式的。应该会在这一步上完成解封装的过程。
2. 音视频分离：对于比如mp4这一类的封装媒体文件，它是一个输入，同时具备音频和视频的，它将在接下来的处理流程上，分为多个stream，在ffmpeg的输出上可以看到Input #n下的Stream #n:m，这就是分离出来的各个流，然后接下来是我们要对这每一个stream进行处理，比如转码等。
3. 解码：如果我们有需要进行音视频处理，比如为音频增加回声，那么将可能会在这一个环节上，完成从封装格式到原始音视频数据解码的过程，为接下来的音频滤镜处理提供数据支持。
4. 滤镜处理：在这一环节上，可以对每一个流进行处理，比如音频可以增加回声，视频可以增加图像处理滤镜等。
4. 编码：可能会在这一步进行A编码到B编码的转换过程。
5. 音视频整合：在这一步完成流的合并的过程。
6. 输出：ffmpeg的输出十分的强大，它跟输入差不多，输出需要声明封装形式，输出的目标，可以是一个或是多个，然后可以是文件、stdout、网络协议等等等等。

> 在上面各环节里，ffmpeg所支持的比如demuxer、encoder、format、协议等都有哪些，可以通过ffmpeg -xxxxs来查看，比如我们看有哪些支持的codec，可以在命令行里输入ffmpeg -codecs，一般输出如下：我们来学习一下如何理解这个输出形式：

```plaintext
Codecs:
 D..... = Decoding supported
 .E.... = Encoding supported
 ..V... = Video codec
 ..A... = Audio codec
 ..S... = Subtitle codec
 ...I.. = Intra frame-only codec
 ....L. = Lossy compression
 .....S = Lossless compression
 -------
 D.VI.S 012v                 Uncompressed 4:2:2 10-bit
 D.V.L. 4xm                  4X Movie
 D.VI.S 8bps                 QuickTime 8BPS video
 .EVIL. a64_multi            Multicolor charset for Commodore 64 (encoders: a64multi )
 .EVIL. a64_multi5           Multicolor charset for Commodore 64, extended with 5th color (colram) (encoders: a64multi5 )
```

在下边输出的第一行里，我们分为三列，第一列为codec的特性编码，第二列为codec名称，第三列为codec的详细说明。然后上边的第一大段落，说明了codec特性编码的含义，D表示解码支持、E表示解码支持，V/A/S表示是音频还是视频或字幕，I表示仅I祯支持，L表示有损压缩，S表示无损压缩。第一列是为了让我们能够快速找到所需要的codec而准备的，知道了这一点，那么我们就可以通过`ffmpeg -codecs | grep DEV..S`，来找到支持视频编解码的无损压缩codec了。对于其它的比如`ffmpeg -formats`、`ffmpeg -protocols`等形式的支持特性的搜索，也是一样的道理。

### codec编码器
这一块不细说了，自己通过`ffmpeg -codecs`查看即可，可以说，ffmpeg支持几乎所有音视频的编解码，非常的全面。

### format封装格式
这一块也不细说了，自己通过`ffmpeg -formats`查看即可。

### protocol协议支持
ffmpeg支持从网络协议上读取或输出它的处理结果，比如`ffmpeg -f mp4 -i tcp://192.168.0.123:111 ...`，它将创建TCP连接到目标服务器的指定端口，并读取需要处理的数据。

输入支持：async、cache、concat、crypto、data、ffrtmpcrypt、ffrtmphttp、file、ftp、gopher、hls、http、httpproxy、https、mmsh、mmst、pipe、rtmp、rtmpe、rtmps、rtmpt、rtmpte、rtmpts、rtp、sctp、srtp、subfile、tcp、tls、udp、udplite、unix、srt。

输出支持：crypto、ffrtmpcrypt、ffrtmphttp、file、ftp、gopher、http、httpproxy、https、icecast、md5、pipe、prompeg、rtmp、rtmpe、rtmps、rtmpt、rtmpte、rtmpts、rtp、sctp、srtp、tee、tcp、tls、udp、udplite、unix、srt

### filter滤镜支持
ffmpeg支持对输入的音频或视频进行各种各样的处理或变换，比如对音频增加回声等，大家可以通过`ffmpeg -filters`来查看其所支持的滤镜，以及各滤镜的说明与用途。

### 命令行实例剖析
```plaintext
ffmpeg
    -f h264 -i video.h264                   # 从video.h264文件中读取，封装形式为h264
    -f s16le -ar 8000 -ac 1 -i audio.pcm    # 从audio.pcm文件中读取，PCM_S16LE编码，8000采样，单声道
    
    -vcodec copy                            # 视频编码直接复制
    -acodec aac                             # 音频编码为AAC
    -f flv                                  # 封装形式flv
    output.flv                              # 输出至目标文件output.flv中去
```
ffmpeg使用参数`-i`来标识输入源，输出没有前置参数，在输入前的参数是用于描述输入的，在输出前的参数是为了修饰输出要求的，所以比如我们要从TCP连接中或是从stdin中读取数据时，一般需要明确声明输入的数据封装形式，比如：`ffmpeg -f h264 -i tcp://localhost:1234...`。


## 其它
### 设备
ffmpeg支持直接从摄像头或麦克风读取音视频数据作为输入。

在windows下，我们可以直接通过`ffmpeg -list_devices true -f dshow -i dummy`来列出所有支持读的设备信息。
```plaintext
> ffmepg -list_devices true -f dshow -i dummy

[dshow @ 0000022d45d7a940] DirectShow video devices (some may be both video and audio devices)
[dshow @ 0000022d45d7a940]  "Logitech HD Webcam C270"
[dshow @ 0000022d45d7a940]     Alternative name "@device_pnp_\\?\usb#vid_046d&pid_0825&mi_00#7&30316c04&0&0000#{65e8773d-8f56-11d0-a3b9-00a0c9223196}\{bbefb6c7-2fc4-4139-bb8b-a58bba724083}"
[dshow @ 0000022d45d7a940] DirectShow audio devices
[dshow @ 0000022d45d7a940]  "楹﹀厠椋?(HD Webcam C270)"
[dshow @ 0000022d45d7a940]     Alternative name "@device_cm_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\wave_{D0016B18-E6B0-4C93-A80D-1065F278F2DF}"
dummy: Immediate exit requested

# 上面会列出所有可用的视频设备和音频视频，我们可以通过设备名称或是别名来让ffmpeg访问到相对应的设备。

# 通过设备名称，读取视频设备的视频数据
> ffmpeg -f dshow -i video="Logitech HD Webcam C270" ...

# 通过设备别名，读取视频设备的视频数据
> ffmpeg -f dshow -i video="@device_pnp_\\?\usb#vid_046d&pid_0825&mi_00#7&30316c04&0&0000#{65e8773d-8f56-11d0-a3b9-00a0c9223196}\{bbefb6c7-2fc4-4139-bb8b-a58bba724083}"

```

linux下，摄像头或麦克风作为设备文件存在于`/dev/`目录下，视频设备一般命名为`/dev/video0`、`/dev/videoN`这种形式。我们可是以直接以`ffmpeg -re -i /dev/video0 ...`的方式从摄像头中获取视频流。linux下没有接触过音频的录制，在这里就不叙述了。

### ffplay
ffmpeg可以从任何源中读取音视频，并输出到任意目标去，但是有时候我们只是想尝试播放一下音频或视频，这时候我们就需要使用`ffplay`命令了，它可以从任意源中读，然后显示、播放读取到的视频或音频。在尝试播放视频时，将显示一个小的视频窗口（可通过左右方向键快进/快退），而播放音频时，也会有一个小窗口，显示的是音频的频谱图。
注意：此命令行需要依赖于图形界面，并且ffplay不能完成转码的过程，在必要的时候，必须声明输入源的封装和编码等信息。比如，如果要播放一段PCM数据（8000采样，16有符号，单声道），则必须声明所有必需的参数：`ffplay -f s16le -ar 8000 -ac 1 -i xxxx.pcm`。

### ffprobe
如果我们对于一个音视频文件，我们不知道它的封装与编码信息的话，可以通过这个命名去探测，但是它也不是万能的，对于无封装形式的PCM就无法识别。所以想要正确识别到，必须是结构化的媒体数据才行。而且`ffprobe`还提供了大量的参数，用于列出媒体文件的详细信息。

* ffprobe -show_streams input.xxx：显示每一个packet的信息，有时候packet和frame的概念很模糊。。。
* ffprobe -show_frames input.xxx：显示每一个frame的信息，比如h264视频数据，每一个祯就是一个frame。
* ffprobe -show_packets input.xxx：显示每一个流的信息，比如单个mp4文件通常同时具有音频和视频，通过此命令，可以看出有几个stream，每个stream是音频还是视频等信息。

### stdio
ffmpeg支持从进程的**stdin**里读取输入，输出到**stdout**里去，这给予我们一个非常简便的能够集成**ffmpeg**的音视频处理能力的接口，用来整合到我们自己的应用中来。因为**stdin**和**stdout**是纯粹的字节流，在使用时，需要明确声明编码或封装形式。

> ffmpeg -f h264 -i - -f mp4 -

在上面的命令行中，我们使用**-**来占位，**-i**后接**-**表示从**stdin**中读取，输出使用**-**占位，表示将输出到**stdout**中去

注意：因为一个进程只有一个**stdin**和一个**stdout**。


### 集成开发指南
在上一个章节里提到ffmpeg可以在运行时，通过stdin读和输出到stdout中去，这为我们自己的应用，集成音视频处理能力提供了可能性，通常我们如果想要做音视频的编解码、转码、或是流媒体的推拉流等工作，要么是找相关的API接口，要不是去学习海量的音视频、流媒体知识，但是掌握了上面的知识点的话，那么我们可以很方便的完成对ffmpeg功能的封装。

简单而言，就是在我们自己的应用里，开启一个子进程，ffmpeg进程（通过运行时参数）设置stdin输入，输出到stdout，我们准备两个线程分别去向进程的stdin写，从stdout读，准备两个线程的原因是子进程的stdio的读写是阻塞的。这样就能够通过设定不同的运行时参数，来完成不同的音视频处理工作，就可以在不依赖于第三方API接口的情况下，简单可靠的集成音视频处理能力到我们自己的应用里了。

下面以Java语言为例，说明一下，如何通过子进程的方式，集成任意封装的音频转为WAV格式的过程。

```java
public class Transcoder
{
	public static void main(String[] args) throws Exception
    {
    	Process process = Runtime.getRuntime().exec("ffmpeg -i - -f wav -");
        // 创建一个线程DataWriter来写数据至进程的stdin
        new DataWriter(process.getOutputStream()).start();
        
        // 创建一个线程DataWriter来读进程的stdout上输出的数据
        new DataReader(process.getInputStream()).start();
        
        // 等待进程终止
        process.waitFor();
    }
    
    static class DataWriter extends Thread
    {
    	OutputStream stdin;
        public DataWriter(OutputStream stdin)
        {
        	this.stdin = stdin;
        }
        
        public void run()
        {
        	while (...)
            {
            	stdin.write(...);
            }
            // 一旦关闭stdin，子进程将在处理完上面写入的数据后自动退出
            stdin.close();
        }
    }
    
    static class DataReader extends Thread
    {
    	InputStream stdout;
        public DataReader(InputStream stdout)
        {
        	this.stdout = stdout;
        }
        
        public void run()
        {
        	while (true)
            {
            	// 返回-1即表示处理进程已经退出了
            	stdout.read(...);
            }
        }
    }
}
```

> 提示：如果有多路输入，多路输出，可以通过Linux下的FIFO命名管道文件来实现，可以通过linux下的unix域文件来实现（仅限linux系统）。还可以通过tcp://127.0.0.1:1122这样的监听socket来实现（平台不限）。

## 写在最后的话
差不多就这样，先写到这为止了。本文档只是为了讲个大概，说清楚最常用的功能和套路，后面我会偶尔补充一下文档。谢谢阅读。
