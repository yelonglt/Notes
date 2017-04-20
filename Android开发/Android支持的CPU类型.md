Android支持三类处理器(CPU)：ARM、Intel和MIPS。ARM使用最广泛，Intel次之，MIPS采用率最低。总之，ARM现在是赢家而Intel是ARM的最强对手。
###ARM处理器和Intel处理器区别
1. ARM使用精简指令集(RISC-Reduced Instruction Set Computing)，而Intel使用复杂指令集(CISC-Complex Instruction Set Computing)。通俗而言，精简指令集规模较小，更接近原子操作，而复杂指令集规模较大，更加复杂。所谓原子操作，是指每条指令的工作大都可以由处理器在一个操作内完成。
2. ARM从来只是设计低功耗处理器。Intel的强项是设计超高新能的台式机和服务器处理器。

###Android系统目前支持的CPU架构
1. ARM处理器。32位(armeabi,armeabi-v7a)，64位(arm64-v8a)。
2. Intel处理器。32位(x86)，64位(x86_64)。
3. MIPS处理器。32位(mips)，64位(mips64)。

###AndroidStudio工程支持不同平台的.so文件的配置
假如app支持armeabi-v7a和x86架构，然后新增一个函数库的依赖，这个函数库包含.so文件并支持更多的CPU架构，编译生成的包在有些机器上会crash。解决方案是重新编译我们的.so文件使其支持缺失的ABIs或者配置gradle文件

```
defaultConfig {
        applicationId "com.wm.dmall"
        minSdkVersion 16
        targetSdkVersion 19
        versionCode 3400
        versionName "3.4.0"
        multiDexEnabled true
        ndk {
            abiFilters "armeabi", "mips", "x86", "armeabi-v7a"
        }
    }
```
###Android只提供ammeabi架构的.so文件好处和坏处
1. 所有的x86、x86_64、armeabi-v7a、arm64-v8a设备都支持armeabi架构的.so文件，因此似乎移除其他ABIs的.so文件是一个减少APK大小的好技巧。但事实上并不是：这不只影响到函数库的性能和兼容性。
2. x86设备能够很好的运行ARM类型函数库，但并不保证100%不发生crash特别是对旧设备。64位设备（arm64-v8a、x86_64、mips64）能够运行32位的函数库，但是以32位模式运行，在64位平台上运行32位版本的ART和Android组件，将丢失专为64位优化过的性能（ART，webview，media等等）。

###查看手机CPU信息
1. adb shell
2. cd /proc
3. cat cpuinfo