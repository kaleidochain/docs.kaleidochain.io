---
title: "iOS版本编译"
weight: 3
---

### 环境准备

可参阅Cocos Creator官方文档<a href="https://docs.cocos.com/creator/2.1/manual/zh/publish/setup-native-development.html" target="_blank">《安装配置原生开发环境》</a>安装XCode。

### 构建

1. Demo源代码下载并解压后，点击Cocos Creator菜单栏的 <b>项目</b> -> <b>构建发布</b> -> <b>构建</b>，Cocos Creator将自动生成iOS版本与Android版本的相关工程文件与代码。

2. 将build_kaleido/jsb-link/frameworks/runtime-src/proj.ios_mac/ios/NativeGengine.h、build_kaleido/jsb-link/frameworks/runtime-src/proj.ios_mac/ios/NativeGengine.m和build_kaleido/jsb-link/frameworks/runtime-src/proj.ios_mac/ios/RootViewController.mm拷贝至build/jsb-link/frameworks/runtime-src/proj.android-studio/ios目录中，并拖动至XCode开发环境中的ios目录。

3. 将build_kaleido/jsb-link/frameworks/runtime-src/proj.ios_mac/ios/portal.lua.h拷贝至build/jsb-link/frameworks/runtime-src/proj.android-studio/ios目录中，并拖动至XCode开发环境中的Resources目录。

4. 将 <a href="https://github.com/kaleidochain/doudizhu/raw/master/KaleidoDoudizhuDemo/build_kaleido/jsb-link/frameworks/runtime-src/proj.ios_mac/Gengine.framework.zip">Gengine.framework</a> 拷贝至build/jsb-link/frameworks/runtime-src/proj.ios_mac/目录中，并拖动至XCode开发环境中的Frameworks目录。

5. 点击工程设置中 <b>Build Phases</b> -> <b>Compile Sources</b> ，在RootViewController.mm后面的Compiler Flags中添加 <b>-fmodules -fcxx-modules</b> 来消除<font color="red">Use of '@import' when modules are disabled</font>的报错。

6. 点击工程设置中 <b>Build Settings</b>，在搜索框中输入CLang，<b>Apple Clang - Language - Modules</b> -> <b>Enable Modules (C and Objective-C)</b> 更改为 <b>Yes</b> 来消除<font color="red">build/jsb-link/frameworks/runtime-src/proj.ios_mac/Gengine.framework/Headers/Universe.objc.h:20:37: No type or protocol named 'goSeqRefInterface'</font>的报错。

7. 上述2至6修改后的工程文件可参考 <b>build_kaleido/jsb-link/frameworks/runtime-src/proj.ios_mac/KaleidoDoudizhuDemo.xcodeproj/project.pbxproj</b>

### 编译与运行

#### 1. iOS Simulator运行

点击Cocos Creator菜单栏的 <b>项目</b> -> <b>构建发布</b> -> <b>编译</b>，编译完成后点击 <b>运行</b>

#### 2. 真机运行

构建完成后使用XCode打开 <b>./build/jsb-link/frameworks/runtime-src/proj.ios_mac/KaleidoDoudizhuDemo.xcodeproj</b>，修改 <b>Bundle Identifier</b> 与 <b>Signing</b>，选择已连接至Mac上的手机后点击XCode中的 <b>运行</b> 按钮即可。
