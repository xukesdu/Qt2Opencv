# Qt2Opencv
Qt配置Opencv,并在Qt中正常使用的步骤
## 2020/11/2 ---配置过程更新
整理时发现Qt官方其实提供了[配置流程](https://wiki.qt.io/How_to_setup_Qt_and_openCV_on_Windows)
---
## 2020/10/27 --- 配置过程
### 一、准备Qt，Opencv，Cmake（顺序为我配置过程安装先后顺序）
---
#### 1.1 Qt的安装
我使用的Qt版本为Qt 5.13.0开源版本，安装过程非常容易，按照步骤一步步来不会有错，<br>其中需要注意在`选择组件`中必须包含`MinGW7.3.0 32-bit`和`MinGW7.3.0 64-bit`，当然我建议电脑空间足够的话可以全选安装。
#### 1.2 Opencv的安装
在安装好Qt后，进行Opencv库的下载安装，我下载的版本为4.5.0，可以直接在[官网](https://opencv.org/releases/ "https://opencv.org/releases/")下载，下载source后直接解压到路径1，路径1我们后面Cmake后用到。
#### 1.3 Cmake的安装
我使用的版本是cmake-3.18.4，下载好后直接运行.exe文件，一步步顺序点击，到`Install Options`时要选择第二项`Add CMake to the system PATH for all users`。<br>如果此处你没有使用软件自带的系统环境配置，请在1.4中自行手动配置。
#### 1.4 系统环境变量的配置
整个配置过程需要配置3大次，Qt，CMake配置在此说明，Opencv编译后配置将在xxx说明。`我的电脑 -> 右键属性 -> 高级系统设置 -> 环境变量 -> 系统变量PATH -> 编辑 -> 新建`依次添加`..\Qt5.13.0\5.13.0\mingw73_32\bin; ..\Qt5.13.0\5.13.0\mingw73_64\bin; ..\Qt5.13.0\tool\mingw730_32\bin; ..\Qt5.13.0\tool\mingw730_64\bin`，其中`..\代表你的安装目录`。CMake如果安装时没有配置好，请参照前面自行配置。
### 二、Opencv动态链接库的建立
#### 2.1 CMake编译Opencv
打开CMake，`Where is the source code`填写路径1，`Where to build the binaries`填写一个你创建的文件夹，用以保存编译后的动态库。<br>
选择完后单击`Configure`会弹出对话框，对话框下拉菜单请选择`MinGW Makefiles`，选择按钮处选择第二个`Specify native compilers`，再点击Next。
会继续弹出对话框，在`C`处选择Qt中gcc.exe的路径（类似于E:/Mysoftware/Qt5.12.22/Tools/mingw730_32/bin/gcc.exe），`C++`选择g++.exe的路径，`Fortran`不填，选择后点击Finish完成。<br>
过一会后会出现一大片红色的，下拉选择`WITH_QT`，取消选择`OPENCV_ENABLE_ALLOCATOR_STATS`,其他默认，点击Configure。完成后会出现几行红色，都是有关Qt路径，检查路径是否有错，修改正确后再次点击Configure，红色完全消失后，点击Generate后完成。（`该过程多注意是否有错，国内一般多为ffmpeg下载不成功，遇到这种情况直接检查创建文件下的CMakeDownloadLog.txt，这里面会提供下载地址，科学上网后可以很快下载好`）。<br>
#### 2.2 编译和安装
在创建文件下`shift+鼠标右键`选择Powershell进入当前文件，输入`mingw32-make`编译，期间我遇上几次错误，总结一下几点：OPENCV_ENABLE_ALLOCATOR_STATS要是不选中状态下进行CMake的；选择的gcc和g++应该为32版本的。`mingw32-make`大约十几分钟，100%后再输入`mingw32-make install`完成安装。`请注意：`安装完成后添加`新建文件夹路径\install\x84\mingw\bin`到系统环境变量。
### 三、在Qt中使用Opencv
#### 3.1使用注意事项
需要在.pro文件中添加
```cpp
INCLUDEPATH += E:\OpenCV\mybuild\install\include

LIBS += E:\OpenCV\mybuild\install\x86\mingw\bin\libopencv_core346.dll
LIBS += E:\OpenCV\mybuild\install\x86\mingw\bin\libopencv_highgui346.dll
LIBS += E:\OpenCV\mybuild\install\x86\mingw\bin\libopencv_imgcodecs346.dll
LIBS += E:\OpenCV\mybuild\install\x86\mingw\bin\libopencv_imgproc346.dll
LIBS += E:\OpenCV\mybuild\install\x86\mingw\bin\libopencv_features2d346.dll
LIBS += E:\OpenCV\mybuild\install\x86\mingw\bin\libopencv_calib3d346.dll
```
添加后运行大概率会crash，我找了一晚上，终于把这些..\mingw\bin\*.dll全部移动到project文件的debug目录下解决了（路径类似于E:\Qtproject\build-opentest-Desktop_Qt_5_13_0_MinGW_32_bit-Debug\debug）。<br>
正确的做法其实是在`.pro`文件里正确配置LIBS，直接把官方代码贴上作为参考：
```cpp
#LIBS += -L$$(OPENCV_SDK_DIR)/x86/mingw/lib \
#        -lopencv_core320        \
#        -lopencv_highgui320     \
#        -lopencv_imgcodecs320   \
#        -lopencv_imgproc320     \
#        -lopencv_features2d320  \
#        -lopencv_calib3d320
```
### 参考与感谢
我配置过程中参考了论坛和网站中各位前辈的配置过程，其中主要有：
* [https://wiki.qt.io/How_to_setup_Qt_and_openCV_on_Windows](https://wiki.qt.io/How_to_setup_Qt_and_openCV_on_Windows) `Qt官方`
* [http://baijiahao.baidu.com/s?id=1662883232064729369&wfr=spider&for=pc](http://baijiahao.baidu.com/s?id=1662883232064729369&wfr=spider&for=pc) `程序客`
* [https://www.cnblogs.com/muyueshi/p/11771716.html](https://www.cnblogs.com/muyueshi/p/11771716.html) `木月石`
* [https://blog.csdn.net/qq_17550379/article/details/78296057](https://blog.csdn.net/qq_17550379/article/details/78296057) `coordinate_blog`
及其他程序员博主，对此我表示由衷的感谢！
