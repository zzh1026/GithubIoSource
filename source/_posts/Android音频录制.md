---
title: Android音频录制
comments: false
date: 2018-05-23 14:12:54
tags:
categories:
    - Android
    - 知识总结

---

# 背景: #

现在几乎所有app都需要录音,所以前些天因为ios和android的不同步问题所以需要把声音录制重新梳理一遍,把自己所理解的一些东西记录一下.


<!-- more -->

## Android使用MediaRecorder录制aac格式音频 ##

    File tempFile = new File(path, "voice.aac");
    audioFile = tempFile.getAbsolutePath();
    mRecorder = new MediaRecorder();
    // 1.设置音频源
    mRecorder.setAudioSource(MediaRecorder.AudioSource.MIC);
    // 2.设置输出格式
    mRecorder.setOutputFormat(MediaRecorder.OutputFormat.AAC_ADTS);
    // 3.设置音频编码器
    mRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AAC);
    // 4.设置输出文件
    mRecorder.setOutputFile(audioFile);
    // 5.调用prepare状态机方法
    try {
       mRecorder.prepare();
    } catch (IllegalStateException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
    // 6.开始录制
    mRecorder.start();
    
    
    // 1.停止录制
    mRecorder.stop();
    // 2.重置
    mRecorder.reset();
    // 3.释放MediaRecorder对象
    mRecorder.release();
    // 4.置空
    mRecorder = null;

详见本人以前的csdn: [android声音录制: https://blog.csdn.net/zzh1026/article/details/52190525](https://blog.csdn.net/zzh1026/article/details/52190525)