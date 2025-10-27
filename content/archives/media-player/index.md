---
categories:
- 默认分类
date: '2025-10-27 17:59:24+08:00'
description: ''
draft: false
image: ''
slug: media-player
cover: /archives/media-player/xv2y7b.png
tags:
- vlc
- player
title: 基于vlc的Player的构建编译
---

## 前言

目前流行的播放器无疑是 PotPlayer 和 VLC，其中 PotPlayer 是韩国公司 Kakao Corp 开发，其开发者是曾是著名的 KMPlayer 的原始作者之一

PotPlayer 并不是开源的软件， VLC 是开源的，并且提供了全平台版本的下载，官方地址在这里 `https://www.videolan.org/`

基于 vlc 的 c++ 开发，需要用到的有两部分：sdk 库文件，以及 libvlcpp 头文件

## sdk库文件

官方首页的播放器下载，并没有提供单独 sdk 文件，需要下载 7z 压缩的播放器，然后解压提取里面的 sdk 目录

下载页面地址在这里 `https://www.videolan.org/vlc/download-windows.html`

![](/archives/media-player/xv2y7b.png)

然后解压 7z 文件，提取里面的 sdk 目录

![](/archives/media-player/im39j9.png)

## libvlcpp 

libvlcpp 主要提供若干 hpp 头文件，并没有需要集成的库文件，在使用的时候，只需要在项目中把头文件 include 进来就可以了

libvlcpp 的下载地址在这里 `https://code.videolan.org/videolan/libvlcpp.git`

项目中也有一些 examples 以及 test 代码可供参考

![](/archives/media-player/pzrxtv.png)

## 构建播放器

新建一个 Player 项目，使用 CMakeList.txt 进行构建，项目结构如下图 

inlude\vlcpp 为前面提到的 libvlcpp 的头文件，include\vlc 和 libs 为 sdk 的部分，main.cpp 为来自 libvlcpp 中的示例代码

![](/archives/media-player/f39h3w.png)

CMakeList.txt 部分代码如下 

```shell
cmake_minimum_required(VERSION 3.20)

project(Player VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

include_directories(${PROJECT_SOURCE_DIR}/include)

file(GLOB_RECURSE SRC_FILES ${PROJECT_SOURCE_DIR}/src/*.cpp)

add_executable(${PROJECT_NAME} ${SRC_FILES})

if(MSVC AND CMAKE_SIZEOF_VOID_P EQUAL 4)
    target_link_options(Player PRIVATE "/SAFESEH:NO")
endif()

target_link_libraries(Player PRIVATE
    ${PROJECT_SOURCE_DIR}/libs/libvlc.lib
    ${PROJECT_SOURCE_DIR}/libs/libvlccore.lib
)

if (MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE
        /W4
        /permissive-
        /utf-8
    )
endif()

install(TARGETS ${PROJECT_NAME} DESTINATION bin)
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/ DESTINATION include)
```

然后在命令行终端中，执行以下命令

```shell
cmake .. -G "Visual Studio 17 2022" -A Win32
cmake --build . --config Release
```

编译后生成 Player.exe 执行文件，程序运行的时候依赖 libvlc.dll 和 libvlccore.dll，把前面播放器里面的动态库拷贝过来即可

还需要把播放器中的 plugins 目录也拷贝过来，运行的时候也依赖这些，然后 `Player.exe bee.mp4` 就可以播放视频了

现在已经是一个完整的播放器了，有画面有声音，运行效果如下

![](/archives/media-player/v6xwz4.png)
