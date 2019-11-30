在公司开始实习，避免每天感到空虚，记录一下每天的学习过程吧

到公司老板给的第一个学习任务，让我优化疲劳驾驶的项目代码，顺便在Linux环境下重构...



# 11.28（星期四）

## 碎碎念：

今天是实习的第四天，周围都是强悍的同事，所幸他们都十分友善~~

研究研究以前一直感兴趣的markdown格式编写吧，从今天开始试着用这个写写日记



## Perf -- Linux性能优化工具

公司是算法公司，所以学一学这些代码热点分析工具吧

------



##  使用git

虽然以前学过这个，但是一直没机会用呀，现在慢慢熟悉吧，从上传这个日记开始~~

翻了翻以前收集的网站，总算把基本操作都熟悉了...

##  计算代码运行时间（opencv）

```c++
#include <opencv2/opencv.hpp>
#include "opencv2/imgproc/imgproc.hpp"
int64 startTime = cv::getTickCount();

//待计算运行时间的函数或代码段

double duration = (cv::getTickCount() - startTime) / cv::getTickFrequency();  //duration即为所计算运行时间
```

虽然visual studio 上有相应的工具，但是项目里有个动态库无法分析，一分析就报错，@#￥%……&*，只好用这个了

用这个方法，终于定位到消耗程序运行时间最多的代码段了...



# 11.29（星期五）

一天了...整整一天了...在**Ubuntu**上装**opencv**把我整疯了！！！，能编译但就是不能运行？？

```c
terminate called after throwing an instance of 'cv::Exception'
  what():  OpenCV(3.4.4) /home/gec/Desktop/sources/modules/highgui/src/window.cpp:698: error: (-2:Unspecified error) The function is not implemented. Rebuild the library with Windows, GTK+ 2.x or Carbon support. If you are on Ubuntu or Debian, install libgtk2.0-dev and pkg-config, then re-run cmake or configure script in function 'cvWaitKey'
```

我早就装了**libgtk2.0-dev  pkg-config**这两个玩意了好吧，整的我重装了好几次**opencv**，还是装不好，我去你妈的！还有用**cmake-gui**装**opencv**，你他们没事链接我的arm qt 库干嘛？？？？

------

拖了两天，总算开始写论文前三章，话说这老师从来不过问我们的毕业设计和毕业论文，~~真是高傲~~，也不知道在想什么...

------

今天终于把疲劳驾驶那个项目最消耗性能的代码层层定位出来了，不能用工具，只能一行行用函数去测试，编译还十分耗时，真是的。。定位之后发现果然不是我能优化的，老板到底在想什么呢...算法还是模型之类的我也不太懂，我只是C++开发工程师啊（这项目写的代码什么注释都没有，代码格式也乱的一批，哪个傻逼写的）



# 11.30(星期六)

公寓里卫生间除了洗手盆，居然没有其他水龙头了。。。于是昨晚买了个小盆，正好能放得下洗手盆，终于可以洗些小衣物了，有点开心

------

一大早出门来公司，发现没开门，原来大家都不加班呐...于是又走了回去...半路又遇到了带了公司钥匙的同事，于是又走回公司了......第一次加班来之前果然应该先跟同事先打声招呼

------

之前装的**opencv3.4**版本一运行就报错，于是装了最新版的...又编译了好一会...配置好后，发现还是这个错误，我要吐了。

我哭了，花了几十分钟，编译到88%，系统cpu资源耗尽卡死了，又重新编译了半小时

------

经过我对**pkg-config**这个软件的深入研究，发现我的系统中该软件默认为交叉编译工具链中内置的工具...也就是说系统一直默认用这玩意在辅助编译x86的**opencv**库？？？于是我追根朔源，终于找到了一直运行报错的原因......居然只是环境变量中交叉编译工具链的路径先于系统默认路径？？？

于是修改**PATH**环境变量中路径的系统默认路径至于交叉编译工具链之前，再次重新编译，终于成功运行**opencv**库函数。这次我真的哭了，我都已经要放弃了。。。