---
title: python 简单的音乐播放器(面向对象思想写的)
date: 2019-03-08 20:16:55
tags: python 练习 面向对象
categories: python加油鸭
---

<!--more-->

# 音乐播放器

```
import os
import pygame
import time

class MusicPlayer():
    musicList = []
    volume = 0.5
    path = r"C:\Users\Administrator\Desktop\music"
    index = 0

    @classmethod
    def getMusicPath(cls):
        fileList = os.listdir(cls.path)
        for filename in fileList:
            absPath = os.path.join(cls.path,filename)
            cls.musicList.append(absPath)

    def __init__(self):
        self.getMusicPath()

    @staticmethod
    def weclome():
        print('''
        **********************
        * 欢迎来到酷我音乐播放器*
        **********************
        ''')

    @staticmethod
    def choice():
        time.sleep(1)
        print('''
        ************************
        * 1.播放       2.停止   *
        * 3.下一曲     4.上一曲  *
        * 5.增大音量   6.减少音量 *
        *       t 退出          *
        *************************
        ''')
        num = input("请选择要操作的功能：")
        return num

    def playMusic(self):
        pygame.mixer.init()
        pygame.mixer.music.load(self.musicList[self.index])
        pygame.mixer.music.play()


    def stopMusic(self):
        pygame.mixer.music.stop()


    def prevMusic(self):
        if self.index == 0:
            print("已经是第一首了")
            return
        else:
            self.index -= 1
            self.playMusic()


    def nextMusic(self):
        if self.index == len(self.musicList)-1:
            print("已经是最后一首了。。。")
            return
        else:
            self.index += 1
            self.playMusic()


if __name__ == "__main__":
    player = MusicPlayer()
    player.weclome()
    while True:
        num = player.choice()
        if num == "1":
            print("播放")
            player.playMusic()
        elif num == "2":
            print("停止")
            player.stopMusic()
        elif num == "3":
            print("下一曲")
            player.nextMusic()
        elif num == "4":
            print("上一曲")
            player.prevMusic()
        elif num == "5":
            print("增大音量")

        elif num == "6":
            print("减少音量")

        elif num == "t":
            break

        else:
            print("选择有误，请重新选择")
            continue
```