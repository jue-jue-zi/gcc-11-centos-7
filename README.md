# gcc-11-centos-7

**注意：**

1. `gcc-11.3.0-built.tar.gz` 中的 `./gcc-temp` 目录为已经编译好的 gcc 11.3。
2. `glibc-2.35-built.tar.gz` 中的 `./build` 目录为已经编译好的 glibc 2.35。
3. `glibc-2.27-built.tar.gz` 中的 `./build` 目录为已经编译好的 glibc 2.27。
4. `make-4.3-built.tar.gz` 中的 `./build` 目录为已经编译好的 make 4.3。

## 编译环境：

```bash
# cat /etc/centos-release
# CentOS Linux release 7.9.2009 (Core)
# uname -r
# 3.10.0-1160.62.1.el7.x86_64
# gcc -v
# gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
```

## 直接使用编译好的版本

```bash
cd /usr/lib64
wget https://raw.githubusercontent.com/jue-jue-zi/gcc-11-centos-7/main/libstdc%2B%2B.so.6.0.29 -O /usr/lib64/libstdc++.so.6.0.29
ln -sf libstdc++.so.6.0.29 libstdc++.so.6
# 运行 clangd 11 12 不需要替换 libm.so
# 不建议替换 libm.so 可能导致以后编译错误
# configure: error: cannot compute sizeof (long long)
wget https://raw.githubusercontent.com/jue-jue-zi/gcc-11-centos-7/main/libm-2.35.so -O /usr/lib64/libm-2.35.so
ln -sf libm-2.35.so libm.so.6
# 如果后续出现编译错误，可以恢复原软链接
# ln -sf libm-2.17.so libm.so.6

# test
wget https://github.com/ycm-core/llvm/releases/download/12.0.0/clangd-12.0.0-x86_64-unknown-linux-gnu.tar.bz2
mkdir clangd-12.0.0
tar xvf clangd-12.0.0-x86_64-unknown-linux-gnu.tar.bz2 -C clangd-12.0.0
./clangd/bin/clangd
```

## 编译命令：

```bash
# ------------------------- gcc -----------------------------

# 编译 gcc 7/11，并替换系统默认 libstdc++
# wget http://mirrors.nju.edu.cn/gnu/gcc/gcc-7.5.0/gcc-7.5.0.tar.gz
wget http://mirrors.nju.edu.cn/gnu/gcc/gcc-11.3.0/gcc-11.3.0.tar.gz
tar -zxvf gcc-11.3.0.tar.gz
cd gcc-11.3.0
yum install -y gcc gcc-c++
# 可以替换 download_prerequisites 中的 base_url='http://www.netgull.com/gcc/infrastructure/' 加速下载
# 并删除 ${fetch} --no-verbose 中的 --no-verbose 以显示下载进度
./contrib/download_prerequisites
mkdir gcc-temp && cd gcc-temp
unset LIBRARY_PATH  # library_path shouldn't contain the current directory when building gcc
unset CPLUS_INCLUDE_PATH  # error "Unable to find a suitable type for HOST_WIDE_INT"
../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
make -j8

# 替换系统 libstdc++
# cp ./x86_64-pc-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6.0.24 /usr/lib64  # gcc 7
cp ./x86_64-pc-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6.0.29 /usr/lib64
cd /usr/lib64
ln -sf libstdc++.so.6.0.29 libstdc++.so.6
# 注意，替换后先新开一个终端，并检查系统程序是否运行正常
# 如果有异常应立刻在原来终端将原软链接恢复

# 查看系统 libstdc++ 支持的版本
strings /usr/lib64/libstdc++.so.6 | grep GLIBCXX

# ------------------------- glibc 2.27 -----------------------------

# activate gcc 7
yum -y install centos-release-scl-rh
yum -y install devtoolset-7-build
yum -y install devtoolset-7-gcc devtoolset-7-gcc-c++
source /opt/rh/devtoolset-7/enable
# gcc -v
# gcc version 7.3.1 20180303 (Red Hat 7.3.1-5) (GCC)

# build glibc
# same as glibc 2.35

# ------------------------- glibc 2.35 -----------------------------

# build make
wget http://mirrors.ustc.edu.cn/gnu/make/make-4.3.tar.gz
tar zxvf make-4.3.tar.gz
cd make-4.3
mkdir build && cd build
../configure  --prefix=/usr/local
make -j12
make install
ln -sf /usr/local/bin/make /usr/bin/make

# activate gcc 8
yum -y install centos-release-scl-rh
yum -y install devtoolset-8-build
yum -y install devtoolset-8-gcc devtoolset-8-gcc-c++
source /opt/rh/devtoolset-8/enable
# gcc -v
# gcc version 8.3.1 20190311 (Red Hat 8.3.1-3) (GCC)

# build glibc
wget https://ftp.gnu.org/gnu/glibc/glibc-2.35.tar.gz
tar zxvf glibc-2.35.tar.gz
cd glibc-2.35
yum -y install bison
mkdir build && cd build
../configure --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
make -j12
cp ./math/libm.so /usr/lib64/libm-2.35.so
cd /usr/lib64
ln -sf libm-2.35.so libm.so.6

strings /usr/lib64/libm.so | grep GLIBC
```
