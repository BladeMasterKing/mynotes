# Tesseract-OCR

## Tesseract-OCR arm架构安装依赖的环境:
- C和C++的编译器：GCC或Clang
- GNU Autotools：autoconf，automake，libtool
- pkg-config
- Leptonica
- libpng，libjpej，libtiff

### 依赖类库
Ubuntu系统需要的类库(Ubuntu 16.04 / 14.04)

```bash
sudo apt-get install g++ # 或者 clang+  系统已安装g++ 5.3.1-1kord
sudo apt-get install autoconf automake libtool # 系统已安装autoconf-2.69-9kord automake-1.15-4kord libtool-2.4.6-0.1kord
sudo apt-get install pkg-conf # 系统已安装 pkg-conf-0.29.1-0kord1
sudo apt-get install libpng-dev  # libpng-dev 未安装
sudo apt-get install libjpeg8-dev # 未安装,支持 libjpeg8-8c-2kord8
sudo apt-get install libtiff5-dev # 未安装,支持 libtiff5-4.0.6-1kord0.6
sudo apt-get install zlib1g-dev # 之前编译安装Nginx中openssl也依赖此类库
```

如果安装OCR的训练工具，还需要其他类库

```bash
sudo apt-get install libicu-dev  # 未安装
sudo apt-get install libpango1.0-dev # 已安装 libpango1.0-dev-1.38.1-1kord
sudo apt-get install libcairo2-dev # 已安装 libcairo2-dev-1.14.6-1kord
```

## 编译Leptonica
Tesseract对应需要安装的Leptonica版本：


Tesseract | Leptonica | ubuntu
--- | --- | ---
4.00 | 1.74.2 | ubuntu 18.04
3.05 | 1.74.0 | 必须从源码构建
3.04 | 1.71 | ubuntu 16.04
3.03 | 1.70 | ubuntu 14.04
3.02 | 1.69 | ubuntu 12.04
3.01 | 1.67 |

下载[leptonica-1.74.4.tar.gz](http://www.leptonica.org/source/leptonica-1.74.4.tar.gz)源码  
构建类库的三种方式:
- 自定义 : 使用已存在的静态makefile(src/makefile.static),通过在(src/environ.h)设置标识来自定义构建。使用leptonica开发建议这种方式
- 使用autoconf : 运行 ./configure 在源码根目录和src目录生成 Makefiles 。Autoconf自动处理以下内容:
    - 架构字节序(endianness)
    - 启用Leptonica的图片IO读写功能(依赖外部扩展库)
    - 启用将格式化的图片IO流重定向到内存  

    执行命令顺序: ./configure --> make && make install。  
    测试命令: make check
- 使用cmake : 构建必须一直在一个不同于源码根目录的目录下执行，通常在根目录创建一个子目录。从根目录执行 mkdir build , cd build , 然后 cmake .. , make  

[更多关于leptonica的内容](http://www.leptonica.org/source/README.html)

### 编译Tesseract
从github上下载源码

```bash
# git clone https://github.com/tesseract-ocr/tesseract.git --branch 4.1 --single-bra
```

确保系统已安装leptonica-1.74.x或更高的版本，然后编译tesseract:

```bash
$ cd tesseract
$ ./autogen.sh
$ ./configure
$ make
$ sudo make install
$ sudo ldconfig
```

### 构建训练工具

```bash
$ cd tesseract
$ ./autogen.sh
$ ./configure
$ make
$ sudo make install
$ sudo ldconfig
$ make training
$ sudo make training-install
```

可以根据需要指定配置选项：

```bash
./configure --disable-openmp --disable-debug --disable-opencl --disable-graphics --disable-shared 'CXXFLAGS=-g -O2 -Wall -Wextra -Wpedantic'
```

### 调试构建
此种构建会产生运行非常缓慢的Tesseract二进制文件。这些文件对生产没用，但是对查找或分析软件问题有用。下面是构建顺序：

```bash
$ cd tesseract
$ ./autogen.sh
$ mkdir -p bin/debug
$ cd bin/debug
$ ../../configure --enable-debug --disable-shared 'CXXFLAGS=-g -O0 -Wall -Wextra -Wpedantic -fsanitize=address,undefined -fstack-protector-strong -ftrapv'
# 构建tesseract和训练工具.如果不需要训练工具，执行"make".
$ make training
$ cd ../..
```

这样激活调试代码，不使用共享的Tesseract库(这样使tesseract无需安装可以运行)，禁用编译器优化，启用大量编译器告警并启用多个运行时检查。

### 分析构建
此构建用于调查性能问题。Tesseract的运行速度将会比没有配置文件时慢，但速度是可接受的，构建顺序：

```bash
cd tesseract
./autogen.sh
mkdir -p bin/profiling
cd bin/profiling
../../configure --disable-shared 'CXXFLAGS=-g -p -O2 -Wall -Wextra -Wpedantic'
# Build tesseract and training tools. Run `make` if you don't need the training tools.
make training
cd ../..
```

它不使用共享的Tesseract库，启用概要分析代码，启用编译器优化并启用许多编译器告警。
这个选项通过添加 --enable-debug 并且将 -o2 替换为 -o0 也可用于调试代码.
剖析代码会在tesseract结束时在当前目录产生一个**gmon.out**的文件,GNU gprof用来展示此文件的分析信息。
### 发布量产版本
默认版本会创建一个Tesseract可执行文件，非常适合处理单个图像。然后，Tesseract使用4个CPU内核来尽快获得OCR结果。

对于具有成百上千个图像的批量生产，默认情况很不好，因为多线程执行的开销非常大。最好运行Tesseract的单线程实例，以便每个可用的CPU内核都可以处理不同的映像。构建顺序：

```bash
cd tesseract
./autogen.sh
mkdir -p bin/release
cd bin/release
../../configure --disable-openmp --disable-shared 'CXXFLAGS=-g -O2 -fno-math-errno -Wall -Wextra -Wpedantic'
# Build tesseract and training tools. Run `make` if you don't need the training tools.
make training
cd ../..
```

此禁用的OpenMP（多线程），不使用共享的Tesseract库（这使得tesseract无需安装即可运行），启用编译器优化，禁用errno数学函数的设置（更快执行！）并启用许多编译器警告。

### 受训的数据
下载[Github](https://github.com/tesseract-ocr/tessdata)上受训的数据，放到 /usr/local/share/tessdata

### 测试

```bash
# Tesseract Open Source OCR Engine v4.1.0 with Leptonica
$ tesseract imagename outputbase [-l lang] [-psm pagesegmode] [configfile...]
# ocr基本执行命令,解析内容输出到out.txt文件
$ tesseract myscan.png out
# 指定语言(德语)
$ tesseract myscan.png out -l deu
# 指定多种语言(英语+德语)
$ tesseract myscan.png out -l eng+deu
# hOCR模式，会生成带有每个词坐标的html文件
$ tesseract myscan.png out hocr
# 生成pdf文件
$ tesseract myscan.png out pdf
```