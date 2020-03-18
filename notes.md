

## Linux查看某个库的位置/是否安装

ldconfig -p | grep libnccl



## 生成stub（.grpc.pb.c/h）

cpp版本：（需要指明grpc安装目录下bin/grpc_cpp_plugin）

```shell
protoc --grpc_out=. --plugin=protoc-gen-grpc=/home/licc/tools/grpc/bin/grpc_cpp_plugin book.proto
```

go版本：

```shell
protoc book.proto --go_out=plugins=grpc:.
```



## 解决GRPC无法通信的问题

修改源码/src/core/lib/surface/init.cc，将grpc_tracer_init();一行注释。参考GRPC源码的21280问题



## go语言学习总结

![1576133593149](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\1576133593149.png)

![1576133695389](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\1576133695389.png)

![1576134301690](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\1576134301690.png)



## 错误总结

```c
‘to_string’ was not declared in this scope‘
```

解决办法：to_string是C++11引入的功能，给编译器加上C++11编译支持：

```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
```



## 协调grpc与人脸SDK的protobuf版本问题



1. 使用grpc 1.19.0 版本

2. 修改grpc目录下Makefile文件：

   （1） 将所有有关ruby的行注释（使其不编译ruby插件）

   （2）修改安装目录prefix=/home/licc/...

   （3）下载github上c-ares项目的1_2版本，复制目录下所有内容到grpc根目录下的		third_party/cares/cares路径

   

## 打印彩色进度条

```c++
#define NONE "\033[0m"
#define RED "\033[0;31m"     //红
#define LIGHT_RED "\033[1;31m"
#define GREEN "\033[0;32m"   //绿
#define LIGHT_GREEN "\033[1;32m"
#define BLUE "\033[0;34m"    //蓝
#define LIGHT_BLUE "\033[1;34m"
#define DARY_GRAY "\033[1;30m"      //灰
#define CYAN "\033[0;36m"           //蓝绿
#define LIGHT_CYAN "\033[1;36m"
#define PURPLE "\033[0;35m"         //紫色
#define LIGHT_PURPLE "\033[1;35m"
#define BROWN "\033[0;33m"          //棕色
#define YELLOW "\033[1;33m"         //黄
#define LIGHT_GRAY "\033[0;37m"
#define WHITE "\033[1;37m"          //白

void PrintBar(int cur, int total)
{

​    int a = (float)cur/total * 100;

​    int b = (float)cur/total * 50;

​    cout << YELLOW"[" + string(b, '=') + ">"LIGHT_GREEN + string(50 - b, '-') << YELLOW"]  "LIGHT_PURPLE << a << "%\r"NONE; 

​    fflush(stdout); // 刷新缓冲区

}
```



## cmake 使用笔记

`添加宏:`

```cmake
add_definitions(-DCOMBINED_MODEL)
```

`option选项开关`

```cmake
option(ARM "use arm-linux-gcc/g++" OFF)
if(ARM)
	...
	add_definitions(-DARM)
else(ARM)
    ...
endif(ARM)


#!/bin/sh
cmake -DTEST_DEBUG=ON ..
```

`增加编译选项：`

```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
```

`扫描文件：`

```cmake
# 递归扫描子目录
file(GLOB_RECURSE HEADER_FILES "3rdparty/caffe/*.hpp"

# 非递归扫描
file(GLOB SOURCE_FILES "3rdparty/caffe/*.cpp"
```

`生成Qt 配置文件 .pro`

```shell
# 在任意文件夹下, 使用qmake -project 命令，会生成.pro 配置文件，并递归扫描子目录所有头文件与源文件添加至.pro 中
qmake -project ..
```

`指定交叉编译(ARM)`

```cmake
SET(CMAKE_C_COMPILER /home/licc/tools/arm/5.4.0/usr/bin/arm-none-linux-gnueabi-gcc)

SET(CMAKE_CXX_COMPILER /home/licc/tools/arm/5.4.0/usr/bin/arm-none-linux-gnueabi-g++)
```



## cmake 添加opencv等第三方模块

```cmake
# OpenCV 3
find_package(OpenCV 3 REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})
link_libraries(${OpenCV_LIBS})
```



## cmake 调用Qt框架

完整链接 [QtCreator构建Cmake工程详细说明](https://blog.csdn.net/a844651990/article/details/82593358)

在ip99服务器下，已有qt框架，在**CMakeLists**文件中可直接调用Qt模块：

```cmake
# 无需添加头文件、库文件等其他操作
find_package(Qt5Core)
target_link_libraries(helloworld Qt5::Core)
```

例子：[cmake 构建Qt5工程](https://blog.csdn.net/wingover/article/details/82728230)

`全局控制：`

```cmake
# 无需设置Qt cmake配置文件路径
# set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} /usr/lib64/cmake)

# Find includes in corresponding build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON)
# Instruct CMake to run moc、uic、rcc automatically when needed.
# 全局控制，打开全区moc...
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)
#set(RESOURCE_DIR resources/resources.qrc)

# Find the QtWidgets library
find_package(Qt5Widgets)
find_package(Qt5Core)


# 获取属性
get_target_property(QtCore_location Qt5::Core LOCATION)
message(${QtCore_location})

add_executable(helloworld main.cpp)
#add_executable(traced_model_demo ${MOC} ${UIC} ${CURRENT_HEADERS} ${CURRENT_SOURCES} mainwindow.ui)
# 直接链接即可
target_link_libraries(helloworld Qt5::Widgets Qt5::Core)
```

`目标控制`

```cmake
find_package(Qt5 REQUIRED Widgets)

set(target_name mainwindow)
add_executable(${target_name} main.cpp mainwindow.cpp mainwindow.h mainwindow.ui)
target_link_libraries(${target_name} Qt5::Widgets)


#设置目标关联的*.h, *.cpp 使用 Qt moc进行编译 
set_target_properties(${target_name} PROPERTIES AUTOMOC ON)
#设置目标关联的*.ui 使用 Qt uic进行编译
set_target_properties(${target_name} PROPERTIES AUTOUIC ON)

#跳过不需要使用moc编译的文件。如果觉得麻烦此句可以不写，automoc能根据*.h,*.cpp代码里面的宏(Q_OBJECT;Q_GADGET;Q_NAMESPACE)自动判断是否需要使用moc
set_source_files_properties(main.cpp PROPERTIES SKIP_AUTOMOC ON)



————————————————
版权声明：本文为CSDN博主「乐z者」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/wingover/article/details/82728230
```

使用该方法，从目标（可执行或者库）入手控制文件生成，可控力度更细一些。


上述两种方法：

使用CMake官方Auto方法生成的VS工程，主要会修改三个地方：

1. 会将一个mocs_compilation.cpp添加到工程里面
2. 【附件包含路径】中会添加一句TestWindow_autogen\include_xxx
3. 
    【预先生成事件】中会添加预处理命令，源文件修改后会自动重新生成moc_xxx.cpp文件



## scp命令上传、下载

**上传文件**

```
scp 本地文件地址 云服务器帐号@云服务器实例公网 IP/域名:云服务器文件地址
```

```shell
scp /home/Inmp0.4.tar.gz root@129.20.0.2:/home/Inmp0.4.tar.gz
```

**下载文件**

```
scp 云服务器帐号@云服务器实例公网 IP/域名:云服务器文件地址 本地文件地址 
```

```
scp root@129.20.0.2:/home/Inmp0.4.tar.gz /home/Inmp0.4.tar.gz
```



## 字符串格式控制/扫描-sscanf()函数

```c++
// 从3种格式中提取任意正确的一种
const std::string formats[] = {"%4u-%2u-%2u", "%4u/%2u/%2u", "%4u%2u%2u"};

const int chread = sscanf(sDate.c_str(), formats[i].c_str(), &year, &month, &day);
if (chread == 3)
	break;

ostringstream oss;
// 格式控制
oss << year << "-" << setfill('0') << std::setw(2) << month << "-" << setfill('0') << std::setw(2) << day;
```



## 替换所有子串

```c++
// replace 有异常抛出
std::string replaceAll(std::string subject, const std::string& search,
		const std::string& replace) {
	size_t pos = 0;
	while ((pos = subject.find(search, pos)) != std::string::npos) {
		subject.replace(pos, search.length(), replace);
		pos += replace.length();
	}
	return subject;
}


ModelEdit路径： https://gitee.com/huawain_application/Tools.git

ModelEncrypt路径：https://gitee.com/huawain_application/Tools.git

人脸SDK路径： https://gitee.com/huawain_ai/FaceRecognitionV2.git
```



## 位操作运算符

[位操作基本运算符](https://www.cnblogs.com/Serenaxy/p/10501950.html)

![1578275329108](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\1578275329108.png)



## git 命令

**ssh生成公钥**

> ```shell
>  ssh-keygen -t rsa -C "xxxxx@xxxxx.com" 
> ```

**设置远程仓库使用ssh协议 **

> ```shell
> $ git remote set-url origin git@xxx.com:xxx/xxx.git
> ```

**把已有仓库与远程仓库关联**

> ```shell
> $ git remote add origin git@github.com:michaelliao/learngit.git
> $ git push -u origin master
> 
> # -u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来
> ```



**推送分支**

```shell
 $ git push origin master
```



**分支**

1.  `git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令：

   ```
   $ git branch dev
   $ git checkout dev
   Switched to branch 'dev'
   ```

2. 合并分支

   ``` shell
   $ git merge dev
   ```

   

3. 删除分支

   ``` shell
   $ git branch -d dev
   ```

4. switch 切换分支 创建并切换到新的dev分支

   ``` shell
   $ git switch -c dev
   ```

5. 版本回退 —— `HEAD`指向当前版本

   + 回退一个版本 : `git reset --hard HEAD^`
   + 回退两个版本 ：`git reset --hard HEAD^^`
   + 回退100个版本 ： `git reset --hard HEAD~100`
   + 回退指定版本 ： `git reset --hard commit_id`
   + 查看提交历史 ： `git log`
   + 重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本

   

6. `git diff HEAD -- readme.txt`命令可以查看工作区和版本库里面最新版本的区别



7. `git checkout -- file`可以丢弃工作区的修改，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况
   + 一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
   + 一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。



8. 丢弃暂存区的修改 ： `git reset HEAD <file>`



9. 自定义git

   + 让Git显示颜色 ： `git config --global color.ui true`

   + 配置别名： 

     ```shell
     $ git config --global alias.co checkout
     $ git config --global alias.ci commit
     $ git config --global alias.br branch
     $ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
     ```

   + 