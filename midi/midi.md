# midi文件

<!-- GFM-TOC -->
- [midi文件](#midi文件)
  - [目标](#目标)
  - [格式转化](#格式转化)
  - [midi](#midi)
<!-- GFM-TOC -->

## 目标
* 首先，为什么要了解midi文件？

最近看到别人写的pcm，很有意思，用代码去写[小星星](https://github.com/hajimehoshi/ebiten/blob/main/examples/pcm/main.go)并且播放，挺好玩的，，这里是通过编写音高，然后整一段波形实现的，这里的效果挺好的，但实际上，我自己移植代码发现，播放的声音很刺耳，然后我了解了新的音频格式midi和新的音色文件格式sf2、sf3。

midi实际上可以算作计算机能看懂的五线谱，midi记录一切曲谱的信息比如乐器，调性，音高，文件小巧，可以说midi和曲谱是可以按规则互相转化的，然后mp3，wav等格式只是音频的波形，无法知道什么乐器，什么音。

所以我投身于用代码编写想要的midi文件，然后就在寻找go中处理midi文件的库，发现有一个[go-midi](https://github.com/gomidi/midi)，当时有点不太想用，因为这个库使用了cgo，然后就找不到有go处理midi文件的库，于是我又去python库找，找到一个[mido](https://github.com/mido/mido)，mido挺好的，例子也很人性化，我萌发了写一个go版本的mido，写着写着，随着我对midi文件格式不断地了解，发现一个问题，go-midi是可以制作midi文件的，而且不需要cgo，别人也比我写的好，我仿写mido用了一堆map和属性很多的message，我好像不需要自己再写了，麻了，不过在写的过程中确实了解了许多有关midi的知识。

暂时，自己的仿写的mido可以先放下，现在用go-midi库写，感觉还可以，符合我的要求

## 格式转化
现在，面临一个新的问题，就是midi格式转mp3或者wav，很多不支持midi格式。这时候，有人肯定会说，用某某软件转化就行了。这个，我也知道，问题是我想通过命令行转化，或者通过api处理，辗转反侧，终于发现了一个前人用的midi转wav工具，timidity，因为工具太老了，教程都是2015年之前的，安装的时候疯狂踩坑，安装完成之后timidity还挺好用的。

## midi
参考[浅析MIDI文件格式](zhuanlan.zhihu.com/p/467534699)
