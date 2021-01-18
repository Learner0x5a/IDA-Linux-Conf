# IDA-Linux-Conf
Configure IDA pro linux version to support custom python/pip. 配置linux版IDA，支持自己的python/pip。

Tested on Ubuntu 18.04 64bit.

## 32位环境配置；32bit enviroment
```bash
dpkg --add-architecture i386
apt update
apt dist-upgrade
apt install gcc-multilib g++-multilib dialog apt-utils
apt install lib32ncurses5 lib32z1
apt install lib32stdc++6 build-essential libssl-dev libffi-dev python-dev libssl-dev libgcc1:i386 zlib1g:i386
```

## 使用自己的python/pip；Use custom python/pip
linux版本的ida pro自带的python是没有binary的，只有一个libpython.so.1.0，没有pip，不能安装第三方的模块比如capstone。

网上的解决方法（Solutions from internet）：

  - 没看懂的操作，但对现有方法做了[汇总](https://duksctf.github.io/2017/03/15/Make-IDA-Pro-Great-Again.html)；A summary
  
  - [idalink](https://github.com/zardus/idalink)：看起来很好用，但要求ida版本>7.0；Require IDA version > 7.0
  

  - [Arybo](https://pythonhosted.org/arybo/integration.html)：用rpc重定向至64位的python；Redirect to 64bit python with rpc
  
  - 官方[博客](https://www.hex-rays.com/blog/installing-pip-packages-and-using-them-from-ida-on-a-64-bit-machine/)：太老了；Official solution, maybe too old.

### 编译一个32位的python2.7；Compile a 32-bit python2.7
```bash
apt install zlib1g-dev:i386 libsqlite3-dev:i386 libssl-dev:i386 libreadline-dev:i386 libncurses5-dev:i386 libffi-dev:i386 libbz2-dev:i386 libgcc1:i386 zlib1g:i386 libc6-dev-i386
wget https://www.python.org/ftp/python/2.7.18/Python-2.7.18.tar.xz
tar xvf Python-2.7.18.tar.xz
cd Python-2.7.18
CFLAGS=-m32 LDFLAGS=-m32 ./configure --prefix=/path/to/python27-32 --with-system-ffi --with-ensurepip=install
make -j24
```
可能有部分模块不能编译；There can be some modules that cannot be built

没有Failed to build these modules即可。But we only need to make sure there's no compilation failure, i.e. Failed to build these modules.

Finally, `make install`

### 使用编译得到的pip安装想要的包；Use compiled python-pip to install whatever you want
```bash
export CFLAGS=-m32
export LDFLAGS=-m32
export LD_LIBRARY_PATH=/lib/i386-linux-gnu/:/usr/lib32:$LD_LIBRARY_PATH
/path/to/python27-32/bin/pip install <package>
```
### 使用安装的包；Use installed packages
在你脚本里添加；In your python script, add
```bash
sys.path.append("/root/env-deps/py27-32/lib/python2.7/site-packages") 
sys.path.append("/root/env-deps/py27-32") 
sys.path.append("/root/env-deps/py27-32/lib/python2.7")
```



## Bug: code for hash md5 was not found
ldd查看IDA PRO自带的Python2.7 lib中_hashlib.so，提示缺少libssl.so.0.9.8 libcrypto.so.0.9.8 libpython2.7.so.1.0。

其中libpython2.7.so.1.0在IDA PRO安装目录下，将其拷贝至对应库目录下即可。
```
cp /path/to/ida/libpython2.7.so.1.0 /path/to/ida/python/lib/python2.7/lib-dynload/
```

IDA PRO 6.8自带的python2.7要求openssl版本为32bit 0.9.8，并且openssl1.0.0和0.9.8不兼容，所以不能通过创建软链接的方式解决，需要安装openssl0.9.8版本。

```bash
wget http://snapshot.debian.org/archive/debian/20110406T213352Z/pool/main/o/openssl098/libssl0.9.8_0.9.8o-7_i386.deb
dpkg -i install libssl0.9.8_0.9.8o-7_i386.deb
```
