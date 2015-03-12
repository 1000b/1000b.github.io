---
layout: post
title: WAV和PCM的关系和区别
category: others
tags: others
keywords: WAV,PCM
description:  WAV和PCM的关系和区别
---

## 什么是WAV和PCM？

**WAV：**wav是一种无损的音频文件格式，WAV符合 PIFF(Resource Interchange File Format)规范。所有的WAV都有一个文件头，这个文件头音频流的编码参数。WAV对音频流的编码没有硬性规定，除了PCM之外，还有几乎所有支持ACM规范的编码都可以为WAV的音频流进行编码。

**PCM:**PCM（Pulse Code Modulation----脉码调制录音)。所谓PCM录音就是将声音等模拟信号变成符号化的脉冲列，再予以记录。PCM信号是由[1]、[0]等符号构成的数字信号，而未经过任何编码和压缩处理。与模拟信号比，它不易受传送系统的杂波及失真的影响。动态范围宽，可得到音质相当好的影响效果。

&emsp;&emsp;简单来说：wav是一种无损的音频文件格式，pcm是没有压缩的编码方式。

## WAV和PCM的关系
	
&emsp;&emsp;WAV可以使用多种音频编码来压缩其音频流，不过我们常见的都是音频流被PCM编码处理的WAV，但这不表示WAV只能使用PCM编码，MP3编码同样也可以运用在WAV中，和AVI一样，只要安装好了相应的Decode，就可以欣赏这些WAV了。在Windows平台下，基于PCM编码的WAV是被支持得最好的音频格式，所有音频软件都能完美支持，由于本身可以达到较高的音质的要求，因此，WAV也是音乐编辑创作的首选格式，适合保存音乐素材。因此，基于PCM编码的WAV被作为了一种中介的格式，常常使用在其他编码的相互转换之中，例如MP3转换成WMA。

&emsp;&emsp;简单来说：pcm是无损wav文件中音频数据的一种编码方式，但wav还可以用其它方式编码。

## 将录音写成wav格式的文件

&emsp;&emsp;有时候需要将录音文件保存为wav格式，这需要手动填充wav的文件头信息，整段代码非常简单，大致如下：

	private RandomAccessFile fopen(String path) throws IOException {
        File f = new File(path);

		if (f.exists()) {
			f.delete();
		} else {
			File parentDir = f.getParentFile();
			if (!parentDir.exists()) {
				parentDir.mkdirs();
			}
		}

        RandomAccessFile file = new RandomAccessFile(f, "rw");
        // 16K、16bit、单声道
        /* RIFF header */
        file.writeBytes("RIFF"); // riff id
        file.writeInt(0); // riff chunk size *PLACEHOLDER*
        file.writeBytes("WAVE"); // wave type

        /* fmt chunk */
        file.writeBytes("fmt "); // fmt id
        file.writeInt(Integer.reverseBytes(16)); // fmt chunk size
        file.writeShort(Short.reverseBytes((short) 1)); // format: 1(PCM)
        file.writeShort(Short.reverseBytes((short) 1)); // channels: 1
        file.writeInt(Integer.reverseBytes(16000)); // samples per second
        file.writeInt(Integer.reverseBytes((int) (1 * 16000 * 16 / 8))); // BPSecond
        file.writeShort(Short.reverseBytes((short) (1 * 16 / 8))); // BPSample
        file.writeShort(Short.reverseBytes((short) (1 * 16))); // bPSample

        /* data chunk */
        file.writeBytes("data"); // data id
        file.writeInt(0); // data chunk size *PLACEHOLDER*

        Log.d(TAG, "wav path: " + path);
        return file;
    }

    private void fwrite(RandomAccessFile file, byte[] data, int offset, int size) throws IOException {
        file.write(data, offset, size);
        Log.d(TAG, "fwrite: " + size);
    }

    private void fclose(RandomAccessFile file) throws IOException {
        try {
            file.seek(4); // riff chunk size
            file.writeInt(Integer.reverseBytes((int) (file.length() - 8)));

            file.seek(40); // data chunk size
            file.writeInt(Integer.reverseBytes((int) (file.length() - 44)));

            Log.d(TAG, "wav size: " + file.length());

        } finally {
            file.close();
        }
    }

## 参考资料

- [什么是PCM？它和.wav文件是什么关系？](http://blog.csdn.net/rinuslucky/article/details/1326512)

- [wave文件(*.wav)格式、PCM数据格式](http://www.cnblogs.com/cheney23reg/archive/2010/08/08/1795067.html)

- [将录音文件保存为wav格式](http://blog.csdn.net/wildwolf_007/article/details/6635120#)



