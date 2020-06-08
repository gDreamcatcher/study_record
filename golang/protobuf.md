# protobuf 安装

## Ubuntu

```shell
// 下载 protoBuf：
$ git clone https://github.com/protocolbuffers/protobuf.git
//	安装依赖库
$ sudo apt-get install autoconf  automake  libtool curl make  g++  unzip libffi-dev -y
// 进入目录
$ cd protobuf/ 
// 自动生成configure配置文件：
$ ./autogen.sh 
// 配置环境：
$ ./configure
// 编译源代码(要有耐心！)：
$ make 
// 安装
$ sudo make install
// 刷新共享库 （很重要的一步啊）
$ sudo ldconfig 
// 成功后需要使用命令测试
$ protoc -h 
```

## centos

```shell
# 下载 protoBuf：
git clone https://github.com/protocolbuffers/protobuf.git
yum install -y autoconf automake  libtool
./autogen.sh 
# 配置环境：
./configure
# 编译源代码(要有耐心！)：
make 
sudo make install
# 刷新共享库 （很重要的一步啊）
sudo ldconfig 
# 成功后需要使用命令测试
protoc -h 
```

