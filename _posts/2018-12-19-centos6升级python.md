---
title: centos6 升级python
date: 2018-12-19 17:29:53
categories:
- python
tags:
- python
- elk
---



```sh
yum install  -y wget
```

#### 1.安装步骤

下载源码

```
wget http://www.python.org/ftp/python/2.7.12/Python-2.7.12.tgz
```

在下载目录解压源码

```
tar -zxvf Python-2.7.12.tgz
```

进入解压后的文件夹

```sh
cd Python-2.7.12
```

在编译前先在/usr/local建一个文件夹`python2.7.12`（作为python的安装路径，以免覆盖老的版本，新旧版本可以共存的)

```sh
mkdir /usr/local/python2.7.12
```

，编译前需要安装下面依赖，否则下面安装pip就会出错

```sh
yum install openssl openssl-devel zlib-devel gcc -y
```

安装完依赖后执行下面命令

```sh
vim ./Modules/Setup.dist
```

找到`#zlib zlibmodule.c -I$(prefix)/include -L$(exec_prefix)/lib -lz`去掉注释并保存(即去掉井号)

在解压缩后的目录下编译安装

```sh
./configure --prefix=/usr/local/python2.7.12 --with-zlib
make
make install
```

此时没有覆盖老版本，再将原来`/usr/bin/python`链接改为别的名字

```sh
mv /usr/bin/python /usr/bin/python2.6.6
```

再建立新版本python的软链接

```sh
ln -s /usr/local/python2.7.12/bin/python2.7 /usr/bin/python
```

这个时候输入
`python`
就会显示出python的新版本信息

> Python 2.7.12 (default, Oct 13 2016, 03:17:14)
> [GCC 4.4.7 20120313 (Red Hat 4.4.7-17)] on linux2
> Type “help”, “copyright”, “credits” or “license” for more information.

#### 2.修改yum配置文件

之所以要保留旧版本，因为yum依赖Python2.6，改下yum的配置文件，指定旧的Python版本就可以了。

```sh
vim /usr/bin/yum`，将第一行的`#!/usr/bin/python`修改成`#!/usr/bin/python2.6.6
```

#### 3.安装最新版本的pip

```sh
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```

找到pip2.7的路径

```sh
find / -name "pip*"
```

> 上面的命令输出
> /root/.cache/pip
> 这里省略一堆输出
> /usr/local/python2.7.12/bin/pip
> /usr/local/python2.7.12/bin/pip2
> /usr/local/python2.7.12/bin/pip2.7 #就是这个
> /usr/bin/pip
> /usr/bin/pip2
> /usr/bin/pip2.6

为其创建软链作为系统默认的启动版本（之前有旧版本的话就先删掉`rm -rf /usr/bin/pip`）

```sh
ln -s /usr/local/python2.7.12/bin/pip2.7 /usr/bin/pip
```

看下pip的版本

```sh
pip -V
pip 18.1 from /usr/local/python2.7.12/lib/python2.7/site-packages/pip (python 2.7)
```







