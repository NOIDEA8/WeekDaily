# 歌曲app

## 1.click中的switch-case报错解决

在根目录下的 **gradle.properties** 中添加 **android.nonFinalResIds=false** 就可以了

## 2.同步、异步、回调（callback）

1.同步调用

A调用B，A需等待B执行完后才能继续往下走。A调用B，A需等待B执行完后才能继续往下走。

2.异步调用

A调用B，调用后，A可以立即执行接下来的程序，不用等B（A不需要B的结果）。

3.回调

为什么需要回调？最初的例子中，A等B制作好油条后(make_youtiao(10000)要跑10分钟)，A再卖出去，即A需要得到B的结果，但是A又不想一直等待10分钟来等到B的结果。于是告诉B在炸好油条后再卖掉就行了（通过回调函数），这样A也不用再关心后续的事情。

### 3.Textview相关，如字体样式

博客：[Android入门教程 | TextView简介（宽高、文字、间距）-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1897419)