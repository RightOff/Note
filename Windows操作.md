# Word

## 格式

### Word 转 PDF后目录中显示：错误未定义书签

全选后按住ctrl+F11或者Ctrl+Shift+F9即可

![1684374518503](image/Windows操作/1684374518503.png)

### 插入mathtype公式后行间距变大

取消以下两个对号。

![1706617419949](image/Windows操作/1706617419949.png)

# 文件夹操作

## 批量复制指定文件

<<<<<<< HEAD

### 插入mathtype公式后行间距变大

取消以下两个对号。

### 生成文件清单

点击主页工具栏上的复制路径图标

![1689391896002](image/Windows操作/1689391896002.png)

粘贴到空白文件中

![1689391952522](image/Windows操作/1689391952522.png)

### 批量复制

将已知的文件名复制到excel中第一列的位置

![1689392033729](image/Windows操作/1689392033729.png)

="copy D:\IMAGE\FAIL\"  & A1 & " D:\IMAGE\test"。其中“D:\IMAGE\FAIL\”与“D:\IMAGE\test”替换成自己对应的源文件路径名和目标路径

再复制一次粘贴为值

![1689392614373](image/Windows操作/1689392614373.png)

新建一个记事本复制copy...的内容

最后把“.txt”修改为“.bat”执行该文件

## 移动文件夹时显示文件被占用

查询被占用文件的方法：

打开任务管理器、选择性能、右上角资源管理器、在关联的句柄里搜索文件夹的名字

![1702367416535](image/Windows操作/1702367416535.png)

找到相关占用文件后结束进程就可以了。

# Windows系统

## 关闭Windows自动更新

永久关闭（关闭10年）。

打开cmd，输入以下命令即可

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WindowsUpdate\UX\Settings" /v FlightSettingsMaxPauseDays /t reg_dword /d 3000 /f
```

## 组策略没有权限打开

删除machine中的两个文件

![1702955922697](image/Windows操作/1702955922697.png)

## win11看B站很卡

考虑到题主其他视频网站都不卡，唯有B站卡，猜测是B站近期大力推行HEVC视频编码标准有关，解决方法如下：

1.进入网页版B站，播放任意视频

2.点击右下角设置-更多播放设置

3.将播放策略一栏的“默认”调整为“AV1”或“AVC”

## 右下角显示未激活水印

新建记事本remove.bat，输入以下代码

```
@echo off
taskkill /F /IM explorer.exe
explorer.exe
exit
```

保存后以管理员身份打开，然后重启电脑。

## 微软商店无法下载

清除缓存

```
wsreset.exe
```

# Git

## Git Bash安装

### 1.下载git for windows并安装，除了以下步骤其他不用管。

![1701576248378](image/Windows操作/1701576248378.png)

### 2.设置用户

打开git bash，输入以下命令

```
git config --global user.name "Your Name"（注意前边是“- -global”，有两个横线）
git config --global user.email "email@example.com"
```

### 3.初始化本地仓库

在本地仓库中打开 git bash

```
git init  //本地仓库初始化
```

### 4.新建远程仓库

![1701576483885](image/Windows操作/1701576483885.png)

![1701576488643](image/Windows操作/1701576488643.png)

### 5.配置Git SSH Keys

 `~/.ssh`检查是否有SSH Keys

如果没有，打开Git Bash，创建SSH Key：`ssh-keygen -t rsa -C "你的邮箱"`。

然后显示：

```
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/16627/.ssh/id_rsa):
//操作1：这是输入文件名用来保存 SSH Key代码，直接回车使用默认。
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
//操作2：设置密码，直接回车不设置密码
Your identification has been saved in /c/Users/…/.ssh/id_rsa
Your public key has been saved in /c/Users/…/.ssh/id_rsa.pub
//3.表明已经设置好了
//有个输入地方需要输入yes
```

打开github

![1701576868984](image/Windows操作/1701576868984.png)

![1701576873495](image/Windows操作/1701576873495.png)

![1701576879078](image/Windows操作/1701576879078.png)

`~/.ssh`查看路径

![1701580931194](image/Windows操作/1701580931194.png)

打开id_rsa.pub文件，全选其中的内容粘贴到网页的Key中。

在git bash中输入 `ssh -T git@github.com`查看是否设置成功，成功的输出

```
Hi “用户名”! You’ve successfully authenticated, but GitHub does not provide shell access.
```

如果连接超时，则需要更改端口号， `vim ~/.ssh/config` 写入以下内容：

```
Host github.com
User xxxxqq.com
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```

### 6.本地端仓库配置

```
git init
git remote add + 仓库名字 + 连接地址  //连接远程仓库，写远程仓库名(可自定义，一般为origin)和地址，例如：git remote add Cplus_Study git@github.com:RightOff/Cplus_Study.git

git remote -v	//查看是否添加成功，输出结果如下  //Cplus_Study     git@github.com:RightOff/Cplus_Study.git (fetch)//Cplus_Study    git@github.com:RightOff/Cplus_Study.git (push)
git remote remove + 仓库名字	//删除连接
```

本地仓库是master 分支，需要修改成和远程仓库的main分支相同的名字

```
git branch -m master main
```

### 7.文件上传

主要最后一条就可以了。

```
$ git add +文件名.文件类型 ，将某个文件加到缓存区
$ git add +文件名.文件类型 ... 文件名.文件类型 ，将n个文件添加到缓存区
$ git add xx文件夹/*.html，将xx文件夹下的所有的html文件添加到缓存区。
$ git add *hhh ，将以hhh结尾的文件的所有修改添加到暂存区
$ git add Hello* ，将所有以Hello开头的文件的修改添加到暂存区
git add -u ，提交被修改(modified)和被删除(deleted)文件，不包括新文件(new)
git add .，提交新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件
…
git add -A，提交所有变化。git add前几条都可以记不住，这个必须记住！！！
```

给add的东西加一个备注，你上传到远程仓库之后，修改的文件后边会显示这个备注

```
git commit -m "修改注释"		//不加-m会进入vim编辑器模式，对新手不友好
```

上传操作：

```
git push -u 仓库名称 分支 -f	//第一次上传需要-f,由于本地和远程第一次存在差异，会报错，因此需要强制覆盖远程仓库的数据
git push -u 仓库名称 分支	//之后就不需要加 -f
```

### 8.文件下拉

方法1：`git pull 仓库名称 分支名`

方法2：

```
git fetch	//将数据拉下来，但是没修改本地的commit和文件   
git merge	//改变本地数据
```

### 9.文件克隆

```
git clone 地址
```

## VScode中配置git

![1701583791620](image/Windows操作/1701583791620.png)

![1701583797723](image/Windows操作/1701583797723.png)

将安装目录下cmd/git.exe的路径写入

![1701583815691](image/Windows操作/1701583815691.png)

重启VScode后查看以下内容表示正确安装。

![1701583871033](image/Windows操作/1701583871033.png)

### git很慢解决方法

在提交的时候填写一下提交信息。

![1701584139619](image/Windows操作/1701584139619.png)

## 报错

### port 22: Connection timed out

详细信息：ssh:connect to host github.com port 22: Connection timed out

```
//进入~/.ssh下
cd ~/.ssh

//创建一个config文件
vim config
```

编辑文件内容

```
Host github.com
User git
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443

Host gitlab.com
Hostname altssh.gitlab.com
User git
Port 443
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
```

检查是否成功

```
ssh -T git@github.com	//之后按提示输入yes
```

# 其他

## 批量复制指定文件

### 1.生成文件清单

点击主页工具栏上的复制路径图标

![1689391896002](image/Windows操作/1689391896002.png)

粘贴到空白文件中

![1689391952522](image/Windows操作/1689391952522.png)

### 批量复制

将已知的文件名复制到excel中第一列的位置

![1689392033729](image/Windows操作/1689392033729.png)

="copy D:\IMAGE\FAIL\"  & A1 & " D:\IMAGE\test"。其中“D:\IMAGE\FAIL\”与“D:\IMAGE\test”替换成自己对应的源文件路径名和目标路径

再复制一次粘贴为值

![1689392614373](image/Windows操作/1689392614373.png)

新建一个记事本复制copy...的内容

最后把“.txt”修改为“.bat”执行该文件

## 关闭Windows自动更新

永久关闭（关闭10年）。

打开cmd，输入以下命令即可

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\WindowsUpdate\UX\Settings" /v FlightSettingsMaxPauseDays /t reg_dword /d 3000 /f
```

## 组策略没有权限打开

删除machine中的两个文件

![1702955922697](image/Windows操作/1702955922697.png)

## win11看B站很卡

考虑到题主其他视频网站都不卡，唯有B站卡，猜测是B站近期大力推行HEVC视频编码标准有关，解决方法如下：

1.进入网页版B站，播放任意视频

2.点击右下角设置-更多播放设置

3.将播放策略一栏的“默认”调整为“AV1”或“AVC”

## Vue安装(实验室网页)

1.进入官网下载Node.js

官网：[https://nodejs.org/zh-cn/download/](https://nodejs.org/zh-cn/download/ "https://nodejs.org/zh-cn/download/") ，版本：node-v18.18.0-x64

安装地址自己设，其他全默认。

2.检查是否安装成功，在命令行输入：

```
node -v
npm -v
```

3.安装淘宝的镜像加速：

```
npm install cnpm@7.1.0 -g --registry=https://registry.npm.taobao.org
```

4.安装vue-cli

```
cnpm install vue-cli -g
```

5.打开项目文件，修改package.json文件

添加 `SET NODE_OPTIONS=--openssl-legacy-provider &&`，修改后如下：

```
  "scripts": {
    "serve": "SET NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service serve",
    "build": "SET NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service build",
    "lint": "SET NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service lint"
  },
```

6.卸载当前版本的 `node-sass`和 `sass-loader`。

因为版本和当前的不兼容，安装对应版本的包即可。

```
#卸载node-sas和sass-loader
cnpm uninstall node-sass
cnpm uninstall sass-loader
```

7.运行

```
npm run server
```

xftp7安装、配置

测试

## 移动文件夹时显示文件被占用

查询被占用文件的方法：

打开任务管理器、选择性能、右上角资源管理器、在关联的句柄里搜索文件夹的名字

![1702367416535](image/Windows操作/1702367416535.png)

找到相关占用文件后结束进程就可以了。

## EndNote21安装

[EndNote新手攻略 | 西园公子的科研百宝箱 (zwjjiaozhu.top)](https://study.zwjjiaozhu.top/posts/EndNote.html)

## 远程桌面连接

### 不能用剪切板

cmd中操作如下命令

```
tasklist | findstr "rdp" 	//查看是否有rdp进程
rdpclip.exe	//没有rdp的话，创建rdpclip
taskkill /im rdpclip.exe /f	//强制杀掉rdpclip进程
rdpclip.exe	//重新启动进程
```

### 总是出现内部错误

1. “ Windows + R ”输入“ services.msc ”。
2. 停止Remote Desktop Services。
3. 重启Remote Desktop Services，并设置启动类型为自动。

第二种方法：

![1708922447714](image/Windows操作/1708922447714.png)

![1708922470486](image/Windows操作/1708922470486.png)

## windows下cmake

下载安装cmake和make，用powershell进行编译，cmake每一次都需要加参数 `-G “MinGW Makefiles”`

```
cmake .. -G “MinGW Makefiles”
make
```

其中，“..”为上一级目录的意思，因为我们的CMakeLists.txt在上一级目录，而”MinGW Makefiles“是指定编译器的意思，由于在Windows下，CMake默认生成VS的编译文件，因此我们需要指定编译器。

### 项目目录结构（多平台通用）

```
MyApp/
  ├─ CMakeLists.txt
  ├─ src/
  │   └─ main.cpp
  ├─ include/
  │   ├─ static_lib/
  │   │   └─ StaticLibHeader.h
  │   └─ dynamic_lib/
  │       └─ DynamicLibHeader.h
  ├─ libs/
  │   ├─ static/
  │   │   └─ libStatic.lib
  │   └─ dynamic/
  │       ├─ libDynamic.dll
  │       └─ libDynamic.lib
  ├─ subproject/
  │   ├─ CMakeLists.txt
  │   ├─ src/
  │   │   └─ subproject_main.cpp
  │   └─ include/
  │       └─ subproject/
  │           └─ SubProjectHeader.h
  └─ config.h.in
```

详细介绍：

1. **根目录** ：包含主项目的 `CMakeLists.txt` 文件以及用于在构建时生成配置文件的 `config.h.in` 文件。
2. **src** ：存放项目源代码的目录，这里只有一个 `main.cpp` 文件作为示例。
3. **include** ：包含头文件的目录。这里有两个子目录，一个是 `static_lib`，包含静态库的头文件 `StaticLibHeader.h`；另一个是 `dynamic_lib`，包含动态库的头文件 `DynamicLibHeader.h`。
4. **libs** ：存放静态库和动态库的目录。这里有两个子目录：`static` 和 `dynamic`。`static` 目录中包含一个名为 `libStatic.lib` 的静态库，`dynamic` 目录中包含一个名为 `libDynamic.dll` 的动态库以及其导入库 `libDynamic.lib`。
5. **subproject** ：一个子项目的目录，包含自己的 `CMakeLists.txt` 文件、源代码（`subproject_main.cpp`）以及头文件（`SubProjectHeader.h`）。

### CMakeList脚本示例

```
# 设置 CMake 最低版本要求
cmake_minimum_required(VERSION 3.8)
# 定义项目名称和版本
project(MyApp VERSION 1.0.0 LANGUAGES CXX)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 定义用户可配置的选项
option(ENABLE_DEBUG "Enable debug output" ON)

if(ENABLE_DEBUG)
  add_definitions(-DDEBUG_OUTPUT)
endif()

# 自定义宏：添加 MSVC 常用编译选项
macro(add_msvc_options target)
  if(MSVC)
    target_compile_options(${target} PRIVATE
    /W4                # 设置警告级别为 4
    /WX                # 将警告视为错误
    /MP                # 启用多处理器编译
    /permissive-       # 禁用不严格的语言 conformance
    /Zc:__cplusplus    # 启用正确的 __cplusplus 宏值
    /Zc:inline         # 移除未使用的函数
    /Gm-               # 禁用最小生成（minimal rebuild）
    /EHsc              # 指定异常处理模型
    )
  endif()
endmacro()

# 添加源文件
set(SOURCE_FILES src/main.cpp)

# 生成可执行文件
add_executable(MyApp ${SOURCE_FILES})

# 调用自定义宏，为 MyApp 添加 MSVC 常用编译选项
add_msvc_options(MyApp)

# 为特定目标设置头文件目录
target_include_directories(MyApp PRIVATE include)

# 链接静态库
find_library(STATIC_LIB libStatic.lib PATHS "${CMAKE_SOURCE_DIR}/libs/static")
target_link_libraries(MyApp PRIVATE ${STATIC_LIB})

# 链接动态库
find_library(DYNAMIC_LIB libDynamic.dll PATHS "${CMAKE_SOURCE_DIR}/libs/dynamic")
find_library(DYNAMIC_LIB_IMPORT libDynamic.lib PATHS "${CMAKE_SOURCE_DIR}/libs/dynamic")
target_link_libraries(MyApp PRIVATE ${DYNAMIC_LIB_IMPORT})

# 使用 Windows 的 DLL delay-load 机制
set_target_properties(MyApp PROPERTIES LINK_FLAGS "/DELAYLOAD:libDynamic.dll")

# 根据目标架构定制编译选项和链接选项
if(CMAKE_GENERATOR_PLATFORM STREQUAL "Win32")
  message("Building for Win32 (x86) architecture")
  target_compile_options(MyApp PRIVATE /arch:SSE2)
elseif(CMAKE_GENERATOR_PLATFORM STREQUAL "x64")
  message("Building for x64 architecture")
  target_compile_options(MyApp PRIVATE /arch:AVX2)
else()
  message(WARNING "Unknown architecture")
endif()

# 添加子项目
add_subdirectory(subproject)

# 在构建时生成配置文件
configure_file(config.h.in config.h @ONLY)

# 指定安装规则
install(TARGETS MyApp RUNTIME DESTINATION bin)
install(FILES "${CMAKE_SOURCE_DIR}/libs/dynamic/libDynamic.dll" DESTINATION bin)
```

### 跨平台构建

`CMakeLists.txt`文件示例，以下if判断方法无法在windows上通过make（跨平台构建需要进一步探索）？？

```
cmake_minimum_required(VERSION 3.10)

project(my_project)

if(WIN32)
    set(CMAKE_CXX_COMPILER "MSVC")
elseif(UNIX)
    set(CMAKE_CXX_COMPILER "g++")
endif()

set(CMAKE_CXX_STANDARD 11)

add_executable(hello hello.cpp)
```

使用if语句来判断平台类型，并设置相应的编译器。通过这种方式，可以实现在不同平台下都可以编译和构建项目的目的。
