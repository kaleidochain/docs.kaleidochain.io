---
title: "安卓版本编译"
weight: 5
---

### 环境准备

可参阅Cocos Creator官方文档<a href="https://docs.cocos.com/creator/2.1/manual/zh/publish/setup-native-development.html" target="_blank">《安装配置原生开发环境》</a>安装Android SDK和Android NDK并在Cocos Creator中配置。

### 构建

1. Demo源代码下载并解压后，点击Cocos Creator菜单栏的 <b>项目</b> -> <b>构建发布</b> -> <b>构建</b>，Cocos Creator将自动生成iOS版本与Android版本的相关工程文件与代码。

2. build_kaleido/jsb-link/frameworks/runtime-src/proj.android-studio目录中一些项目文件的内容供开发者们参考，开发者需要根据自己的实际情况进行修改。

3. 将build_kaleido/jsb-link/frameworks/runtime-src/proj.android-studio/app/src/org/cocos2dx/javascript/AppActivity.java中的修改拷贝至build/jsb-link/frameworks/runtime-src/proj.android-studio/app/src/org/cocos2dx/javascript/AppActivity.java

4. 将build_kaleido/jsb-link/frameworks/runtime-src/proj.android-studio/app/src/io整个目录拷贝至build/jsb-link/frameworks/runtime-src/proj.android-studio/app/src/

5. 在build/jsb-link/frameworks/runtime-src/proj.android-studio/app/中创建libs目录，并将 <a href="https://github.com/kaleidochain/doudizhu/raw/master/KaleidoDoudizhuDemo/build_kaleido/jsb-link/frameworks/runtime-src/proj.android-studio/app/libs/gengine.aar">gengine.aar</a> 拷贝到libs目录。

6. 修改build/jsb-link/frameworks/runtime-src/proj.android-studio/app/build.gradle，将 <b>minifyEnabled</b> 和 <b>shrinkResources</b> 均设置为 <b>false</b> 。
```
buildTypes {
        release {
            debuggable false
            jniDebuggable false
            renderscriptDebuggable false
            minifyEnabled false
            shrinkResources false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            if (project.hasProperty("RELEASE_STORE_FILE")) {
                signingConfig signingConfigs.release
            }

            ...
        }
}
```

7. 修改build/jsb-link/frameworks/runtime-src/proj.android-studio/app/proguard-rules.pro，在文件最后添加
```
# keep kaleido for release.
-keep public class io.kaleidochain.** { *; }
-dontwarn io.kaleidochain.**
```

### 编译与运行

点击Cocos Creator菜单栏的 <b>项目</b> -> <b>构建发布</b> -> <b>编译</b>，最后使用 <b>adb install</b> 进行安装运行。