# RT-Thread

## 音频设备驱动开发

很久之前机缘巧合拿到一块万利电子的LPC54114-Lite开发板，一直以来都对mp3念念不忘，之前也在这块板子上跑过libmp3这个开源的解码库来播放mp3，不过那时跑的是freeRTOS，最近查看rt-thread文档时恰巧看到rt-thread文档中包含了audio设备相关的说明，心血来潮，参考其他bsp中已有的audio驱动，做了一些基础的功能，简单实现了一下该板卡上的音频设备驱动。

### 驱动结构
简单看了下audio设备的源码，rt-thread的设备驱动设计思想和Linux接近，提供了一系统通用的ops回调函数，由特定的bsp代码来实现这些ops回调函数，然后用户代码就能使用通用的read/write来操作设备了。这里我们只需要关注

```c
struct rt_audio_ops
{
    rt_err_t (*getcaps)(struct rt_audio_device *audio, struct rt_audio_caps *caps);
    rt_err_t (*configure)(struct rt_audio_device *audio, struct rt_audio_caps *caps);
    rt_err_t (*init)(struct rt_audio_device *audio);
    rt_err_t (*start)(struct rt_audio_device *audio, int stream);
    rt_err_t (*stop)(struct rt_audio_device *audio, int stream);
    rt_size_t (*transmit)(struct rt_audio_device *audio, const void *writeBuf, void *readBuf, rt_size_t size);
    /* get page size of codec or private buffer's info */
    void (*buffer_info)(struct rt_audio_device *audio, struct rt_audio_buf_info *info);
};
```
这个ops结构就好了，每个回调函数的功能从字面意义看已经很明显了，具体的实现我们可以参考其他bsp已有的audio驱动，这里我参考的是qemu-vexpress-a9这个bsp中的audio驱动，其实只要正确的实现`init`、`start`、`stop`、`transmit`、`buffer_info`这几个回调函数，音频设备就可以使用了，只不过无法进行配置。
这里还需要关注的是`rt_audio_tx_complete`这个函数，这个函数会将待发送的数据拷贝至`buffer_info`所指定的缓冲区中，如果transmit这个回调存在的话，会触发该回调进行数据传输。

### ops回调实现
这些ops回调都跟具体的器件和硬件连线相关了，`init`做具体硬件的初始化，`start`使能器件进行数据传输，`stop`结束数据传输，`transmit`开始一个新的数据块传输，`buffer_info`获取缓存区信息，直接参考其他bsp的实现就可以。
需要注意的是，我们一般使用DMA来进行音频数据的传输，这里需要考虑不同mcu的差异，对于stm32f4来说，他的dma支持半空和全空中断，这样就可以将dma配置为乒乓传输模式，可以省去transmit这个回调。
对于LPC54114-Lite这个板卡，他的dma是可以配置成乒乓模式的，但是dma只有全空这个类型的中断，所以这里就暂且实现一下transmit这个回调，由这个回调来触发数据的传输。
基本上到这里音频设备就可以使用了，可以通过一些简单的测试播放固定频率的音频，然后实现`configure`回调之后就可以设置音频采样率这些参数了，音频设备就可以使用了。
