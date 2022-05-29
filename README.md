# gcc-11-centos-7

**注意：`gcc-11.3.0-built.tar.gz` 中的 `./gcc-temp` 目录为已经编译好的 gcc 11.3。**

## 编译环境：

```bash
# cat /etc/centos-release
# CentOS Linux release 7.9.2009 (Core)
# uname -r
# 3.10.0-1160.62.1.el7.x86_64
# gcc -v
# gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
```

## 编译命令：

```bash
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
```
