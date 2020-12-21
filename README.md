## 依赖

#### `Linux`

集成iRRAM的tensorflow可在**ubuntu18.04**环境下进行（已验证可行），其他环境自行探索

#### `bazel`

tensorflow的编译主要依赖bazel工具进行，安装bazel需要手动先安装该工具需要的依赖，对于bazel的安装我们采用shell脚本的方式手动安装。**安装bazel前需要保证环境下已安装JDK（8）**

##### 第1步：安装所需的软件包

```shell
sudo apt-get install pkg-config zip g++ zlib1g-dev unzip python
```

##### 第2步：下载Bazel

集成iRRAM版本的tensorflow需要采用bazel-0.26.1的版本，`bazel-0.26.1-installer-linux-x86_64.sh` 从[GitHub上](https://github.com/bazelbuild/bazel/releases)的[Bazel发布页面](https://github.com/bazelbuild/bazel/releases)下载Bazel二进制安装程序

##### 第3步：运行安装程序

```shell
chmod +x bazel-0.26.1-installer-linux-x86_64.sh
./bazel-0.26.1-installer-linux-x86_64.sh --user
```

##### 第4步：设置您的环境

使用`--user`标志运行Bazel安装程序，则Bazel可执行文件将安装在的`$HOME/bin`目录中，需要将该路径加入环境变量中

```shell
vim ~/.bashrc
# 最后一行加入
export PATH=$PATH:$HOME/bin
# 保存文件后，刷新文件
source ~/.bashrc
```

#### `gmp`

##### 第1步：下载gmp安装包

可从GNU官网下载[gmp二进制包](https://gmplib.org/#DOWNLOAD)，选择gmp-<version>.tar.bz2解压至任意路径下并进入解压目录

##### 第2步：安装

gmp会默认安装至 /usr/local/include 和 /usr/local/lib目录下

```shell
./configure
make
make check
# 上一行无报错可进行下面的安装操作
sudo make install
```

#### `mpfr`

##### 第1步：下载mpfr安装包

可从GNU官网下载[mpfr二进制包](https://www.mpfr.org/mpfr-current/#download)，选择mpfr-<version>.tar.bz2解压至任意路径下并进入解压目录

##### 第2步：安装

mpfr会默认安装至 /usr/local/include 和 /usr/local/lib目录下,mpfr依赖gmp，需要先安装gmp

```shell
./configure --with-gmp-include=/usr/local/include --with-gmp-lib=/usr/local/lib
make
make check
# 上一行无报错可进行下面的安装操作
sudo make install
```

#### `iRRAM`

##### 第1步：下载iRRAM源码

 从[GitHub上](https://github.com/bazelbuild/bazel/releases)的[iRRAM](https://github.com/DJY160017/iRRAM_improved_byron)下载iRRAM源码

##### 第2步：安装依赖

iRRAM工具使用依赖需参见**其它依赖**

##### 第3步：安装iRRAM

iRRAM会默认安装至 /usr/local/include 和 /usr/local/lib目录下

```shell
sudo ./QUICKINSTALL_run_me
# 脚本执行过程中选择安装至系统中，可选择是否编译测试
sudo ldconfig
```

#### `其他依赖`

```shell
sudo apt-get install autoconf automake libtool
```

## 编译tensorflow

#### 第一步：下载tensorflow源码

从[GitHub上](https://github.com/bazelbuild/bazel/releases)的[tennsorflow](https://github.com/DJY160017/tensorflow)下载tensorflow源码,并在本地切换至iRRAM分支

#### 第二步：编译

1. 进入源码根目录执行

   ```shell
   #配置中python选择python3
   ./configure
   ```

2. 进行编译

   ```shell
   # c++版本
   bazel build -c opt --verbose_failures //tensorflow:libtensorflow_cc.so
   ```
#### 第三步：编译其他依赖

进入 tensorflow/contrib/makefile 目录

```shell
./build_all_linux.sh
```

## 测试

提供Cmakelist.txt具体内容进行参考

```
cmake_minimum_required(VERSION 3.15)
# 项目名
project(***)

set(CMAKE_CXX_STANDARD 11)

#需要执行和编译的文件
set(SOURCE_FILES *.cpp)


#引入tensotflow编译结果的根目录，tensorflow的根目录
set(TENSORFLOW_ROOT_DIR */tensorflow)

# 设置源文件的引用目录
include_directories(
        ${TENSORFLOW_ROOT_DIR}
        ${TENSORFLOW_ROOT_DIR}/bazel-genfiles
        ${TENSORFLOW_ROOT_DIR}/bazel-out
        ${TENSORFLOW_ROOT_DIR}/tensorflow/contrib/makefile/gen/proto
        ${TENSORFLOW_ROOT_DIR}/tensorflow/contrib/makefile/gen/host_obj
        ${TENSORFLOW_ROOT_DIR}/tensorflow/contrib/makefile/gen/protobuf/include
        ${TENSORFLOW_ROOT_DIR}/tensorflow/contrib/makefile/downloads/nsync/public
        ${TENSORFLOW_ROOT_DIR}/tensorflow/contrib/makefile/downloads/eigen
        ${TENSORFLOW_ROOT_DIR}/tensorflow/contrib/makefile/downloads/absl
)

#lib
link_directories(${TENSORFLOW_ROOT_DIR}/bazel-bin/tensorflow)

# 设置程序启动入口，后期可能指定某个包
add_executable(tensorflow_demo ${SOURCE_FILES})

# 设置需要链接的tensorflow库
target_link_libraries(***
        mpfr
        gmp
        iRRAM
        ${TENSORFLOW_ROOT_DIR}/bazel-bin/tensorflow/libtensorflow_cc.so
        ${TENSORFLOW_ROOT_DIR}/bazel-bin/tensorflow/libtensorflow_framework.so.2
        )
```

## 其他
tensorflow中的依赖存在失效或者下载网速慢的情况，需要手动下载，搭建文件服务器可成功
