# VS下使用Protocol buffer（C++）

*首先，给出官方的网址，各种官方资料都可以找到https://github.com/google/protobuf*

> windows下使用VS进行Protocol buffer开发（C++）的基本步骤：

1.下载代码包

2.形成sln文件

3.编译出需要的lib文件和protoc.exe(exe程序可以直接下载得到)

4.编写.proto文件

5.利用protoc.exe文件生成.c和.h文件

6.在工程中添加相关资源（.c和.h文件以及lib库）  完成

## 1.下载代码包

*https://github.com/google/protobuf*

## 2.形成sln文件

### （1）2.6.1版本：

​	2.6.1版本中，打开文件夹“vsprojects”,里面有个.sln文件，打开就可以了，对，就是这么简单。

### （2）3.0.0版本

​	3.0.0版本中，没有这个文件夹,cmake文件夹下readme中明确的告诉了我们怎么生成这个sln文件，需要下载Cmake工具

​	打开vs2013 x64本机工具命令提示,在开始菜单里，vs下面的tools里面，我的路径C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\Tools\Shortcuts，实在找不到可以参考一下。

安装或者解压你下载的cmake

在我们亲爱的黑框框里（x64本机工具命令提示） 添加cmake环境变量（path\to 是官方文档里的实例路径） 

```
C:\Path\to>set PATH=%PATH%;C:\Program Files (x86)\CMake\bin
```

cd命令进入你下载源码包的cmake文件夹下，在cmake文件夹中创建新目录build并进入。

```
C:\Path\to\protobuf\cmake>mkdir build & cd build
 C:\Path\to\protobuf\cmake\build>
```

创建一个release目录，如果成功，黑框框中不提示错误，并且你生成了一坨东西在对应的目录下。

```
C:\Path\to\protobuf\cmake\build>mkdir release & cd release
 C:\Path\to\protobuf\cmake\build\release>cmake -G "NMake Makefiles" ^
 -DCMAKE_BUILD_TYPE=Release ^
 -DCMAKE_INSTALL_PREFIX=../../../../install ^
 ../..
```

创建一个debug目录

```
C:\Path\to\protobuf\cmake\build>mkdir debug & cd debug
 C:\Path\to\protobuf\cmake\build\debug>cmake -G "NMake Makefiles" ^
 -DCMAKE_BUILD_TYPE=Debug ^
 -DCMAKE_INSTALL_PREFIX=../../../../install ^
 ../..
```

最关键的一步来啦，生成sln文件

```
C:\Path\to\protobuf\cmake\build>mkdir solution & cd solution
 C:\Path\to\protobuf\cmake\build\solution>cmake -G "Visual Studio 12 2013 Win64" ^
 -DCMAKE_INSTALL_PREFIX=../../../../install ^
 ../..
```

## 3.编译出需要的lib文件

​	之后用vs打开该工程,编译，生成目录里多了东西，其中就有protoc.exe和libprotobuf.lib和libprotoc.lib文件，就是我们需要的

## 4.编写.proto文件

一个简单的例子：

      option optimize_for = LITE_RUNTIME;
      message LogonReqMessage {
          required int64 acctID = 1;
          required string passwd = 2;
      }
## 5.生成.cc和.h文件

```
protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/xxx.proto
```

> 如```protoc -I =. --cpp_out=. ./MyMessage.proto```
>
> 将会在当前目录下生成MyMessage.pb.h和MyMessage.pb.cc文件

## 6.使用的工程中添加相关资源

​	将.cc .h以及libprotobuf.lib，libprotoc.lib拷贝到工程目录下，添加附加包含目录：**项目属性->C++>常规->附加包含目录：E:\protobuf-2.6.1\protobuf-2.6.1\src;%(AdditionalIncludeDirectories)**，然后编译

遇到的问题：

​	（1）error c4996 std::Copy_impl。。。。。。。

​	解决方法：

​	点击“属性”中的“c\c++”下的“预处理器”，在右侧的“预处理器定义”中添加_SCL_SECURE_NO_WARNINGS。

​	（2）无法解析的外部符号

​	解决方法：

​	工程中添加libprotobuf.lib，libprotoc.lib

示例代码：

```c++
include "pb\MyMessage.pb.h"

void main()
{
	printf("==================This is simple message.================\n");
	//序列化LogonReqMessage对象到指定的内存区域。
	LogonReqMessage logonReq;
	logonReq.set_acctid(20);
	logonReq.set_passwd("Hello World");
	//提前获取对象序列化所占用的空间并进行一次性分配，从而避免多次分配
	//而造成的性能开销。通过该种方式，还可以将序列化后的数据进行加密。
	//之后再进行持久化，或是发送到远端。
	int length = logonReq.ByteSize();
	char* buf = new char[length];
	logonReq.SerializeToArray(buf, length);
  
  	//从内存中读取并反序列化LogonReqMessage对象，同时将结果打印出来。
	LogonReqMessage logonReq2;
	logonReq2.ParseFromArray(buf, length);
	printf("acctID = %I64d, password = %s\n", logonReq2.acctid(), logonReq2.passwd().c_str());
	delete[] buf;
  	buf = NULL;
}
```