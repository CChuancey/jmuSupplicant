# jmuSupplicant

[![License](https://img.shields.io/crates/l/rustc-serialize.svg)](https://raw.githubusercontent.com/ShanQincheng/jmuSupplicant/master/LICENSE)

- 本项目基于jmuSupplicant修改完成，适用于常熟理工学院校园网认证。
- 感谢原作者提供源代码
- 很遗憾没能实现11点后断网认证。
- padavan固件下使用不是很稳定，尽量使用op。

# 编译（以Unbuntu 18.04LTS为例）

## 安装所需依赖

> ```bash
> build-essential bison flex zlib1g-dev libncurses5-dev subversion quilt intltool ruby fastjar unzip gawk autogen autopoint
> ```

## 安装libpcap库和cmake工具

- 自行安装

## 普通编译

```bash
git clone https://github.com/MiChuancey/jmuSupplicant.git
cd jmuSupplicant
mkdir build
cd build
cmake ../
make
```

之后可以在 ```build/bin``` 目录下找到 jmuSupplicant 的可执行文件。

## 交叉编译

交叉编译需要先编译 libpcap ，之后再编译 jmuSupplicant。下面以交叉编译到 ar71xx 路由器为例：(以下代码中的一些参数需要根据你的实际情况做相应的修改，仅供参考)

### 获取目标设备的交叉编译工具链

从 [https://downloads.openwrt.org/](https://downloads.openwrt.org/) 上面下载目标设备的交叉编译工具链。例如 ar71xx 芯片的工具链下载地址为：[https://downloads.openwrt.org/releases/18.06.0/targets/ar71xx/generic/openwrt-sdk-18.06.0-ar71xx-generic_gcc-7.3.0_musl.Linux-x86_64.tar.xz](https://downloads.openwrt.org/releases/18.06.0/targets/ar71xx/generic/openwrt-sdk-18.06.0-ar71xx-generic_gcc-7.3.0_musl.Linux-x86_64.tar.xz)

(若下载缓慢，可以到[清华大学镜像源](https://mirrors.tuna.tsinghua.edu.cn/lede/)以及[中国科学技术大学镜像源](https://mirrors.ustc.edu.cn/lede/)下载相应工具链)

```bash
wget https://downloads.openwrt.org/releases/18.06.0/targets/ar71xx/generic/openwrt-sdk-18.06.0-ar71xx-generic_gcc-7.3.0_musl.Linux-x86_64.tar.xz
tar xvJf openwrt-sdk-18.06.0-ar71xx-generic_gcc-7.3.0_musl.Linux-x86_64.tar.xz
```

### 配置环境变量

环境变量中的具体路径以及参数要根据你的实际情况做相应的修改，以下代码仅供参考：

```bash
export PATH=$PATH:/home/xxx/openwrt-sdk-18.06.0-ar71xx-generic_gcc-7.3.0_musl.Linux-x86_64/staging_dir/toolchain-mips_24kc_gcc-7.3.0_musl/bin
export CC=mips-openwrt-linux-gcc
export CPP=mips-openwrt-linux-cpp
export GCC=mips-openwrt-linux-gcc
export CXX=mips-openwrt-linux-g++
export RANLIB=mips-openwrt-linux-ranlib
export LC_ALL=C
export LDFLAGS="-static"
export CFLAGS="-Os -s"
export STAGING_DIR=/home/xxx/openwrt-sdk-18.06.0-ar71xx-generic_gcc-7.3.0_musl.Linux-x86_64/staging_dir/toolchain-mips_24kc_gcc-7.3.0_musl
```

### 交叉编译 libpcap

```bash
wget http://www.tcpdump.org/release/libpcap-1.9.0.tar.gz
tar zxvf libpcap-1.9.0.tar.gz
cd libpcap-1.9.0
./configure --host=mips-linux --with-pcap=linux
make
```

如果交叉编译 libpcap 的过程中遇到错误，不用担心，这里我们只需要用到 ```libpcap.a``` ，编译后能得到该文件即可。之后将该文件以及 libpcap 的相关头文件复制到工具链的目录中：

```bash
cp libpcap.a /home/xxx/openwrt-sdk-18.06.0-ar71xx-generic_gcc-7.3.0_musl.Linux-x86_64/staging_dir/toolchain-mips_24kc_gcc-7.3.0_musl/lib
cp pcap.h /home/xxx/openwrt-sdk-18.06.0-ar71xx-generic_gcc-7.3.0_musl.Linux-x86_64/staging_dir/toolchain-mips_24kc_gcc-7.3.0_musl/include
cp -r pcap /home/xxx/openwrt-sdk-18.06.0-ar71xx-generic_gcc-7.3.0_musl.Linux-x86_64/staging_dir/toolchain-mips_24kc_gcc-7.3.0_musl/include
```

### 交叉编译 jmuSupplicant

```bash
git clone https://github.com/MiChuancey/jmuSupplicant.git
cd jmuSupplicant
mkdir build
cd build
cmake ../ -DCMAKE_FIND_ROOT_PATH=/home/xxx/openwrt-sdk-18.06.0-ar71xx-generic_gcc-7.3.0_musl.Linux-x86_64/staging_dir/toolchain-mips_24kc_gcc-7.3.0_musl -DCMAKE_FIND_ROOT_PATH_MODE_LIBRARY=ONLY -DCMAKE_C_COMPILER=/home/xxx/openwrt-sdk-18.06.0-ar71xx-generic_gcc-7.3.0_musl.Linux-x86_64/staging_dir/toolchain-mips_24kc_gcc-7.3.0_musl/bin/mips-openwrt-linux-gcc
make
```

之后可以在 ```build/bin``` 目录下找到 jmuSupplicant 的可执行文件。

# 使用说明

可以通过```--help```参数来获取程序运行帮助，下面举例两种使用情况：

## 正常使用

- 使用以下指令进行锐捷认证：

  ```bash
  sudo ./jmu -u 学号 -p 密码 -s0(教育网接入)1(联通宽带接入)2(移动宽带接入)3(电信宽带接入) -b
  ```

- 程序输出锐捷认证信息，或显示 login success， 则认证成功。

# 已测试稳定运行的设备

- 计算机：
  - Ubuntu 18.04
- 路由器：
  - MT7621
  - MT7620

# License

Apache version 2.0
