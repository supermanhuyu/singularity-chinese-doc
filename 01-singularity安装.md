---
title: singularity 安装教程
---

首先是下载源码，singularity下载地址：<http://singularity.lbl.gov/all-releases>

下载到本地之后，解压

```
tar xvf singularity-2.2.1.tar.gz
mkdir /gensoft/singularity
cd singularity-2.2.1/
./configure 
make
make install 
```

到这里其实singularity已经安装完毕，下面是参照官网说明做的一个镜像

```
BootStrap: yum
OSVersion: 7
MirrorURL: http://mirror.centos.org/centos/7/os/x86_64/
Include: yum

```

[yhu@master ~]$sudo 

[root@7-2 singularity-2.2.1]#singularity shell --contain /tmp/Centos7.img

Singularity: Invoking an interactive shell within container...

Singularity.Centos7.img> id

uid=0(root) gid=0(root) groups=0(root)

Singularity.Centos7.img> ls /

bin   dev	   etc	 lib	lost+found  mnt  proc  run   singularity  sys  usr

boot  environment  home  lib64	media	    opt  root  sbin  srv	  tmp  var

Singularity.Centos7.img> exit

exit

singularity制作镜像也可以直接从docker的镜像中获取，使用方法如下：

```
[yhu@master ~]$sudo singularity import /tmp/tensorflow.img docker://tensorflow/tensorflow:latest
tensorflow/tensorflow:latest
Downloading layer: sha256:a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4
...

Adding Docker CMD as Singularity runscript...
Bootstrap initialization
No bootstrap definition passed, updating container
Executing Prebootstrap module
Executing Postbootstrap module
Done.

```

或者

```
Bootstrap: docker
From: tensorflow/tensorflow:latest

%runscript

    exec /usr/bin/python "$@"

%post

    echo "Post install stuffs!"
```

```
#测试结果
[yhu@master ~]$sudo ./singularity shell /tmp/tensorflow.img 
Singularity: Invoking an interactive shell within container...

Singularity.tensorflow.img> ls 
anaconda-ks.cfg       easy-rsa-release-2.x.zip	singularity-2.2.1
easy-rsa-release-2.x  rpmbuild			singularity-2.2.1.tar.gz
Singularity.tensorflow.img> python	    
Python 2.7.12 (default, Nov 19 2016, 06:48:10) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
Singularity.tensorflow.img> pwd 
/root
Singularity.tensorflow.img> exit


```

singularity在官网还有众多详细资料

官方资料：

singularity下载地址：<http://singularity.lbl.gov/all-releases>

安装教程：<http://singularity.lbl.gov/quickstart>

使用docker制作镜像, <http://singularity.lbl.gov/docs-docker#use-a-spec-file>

在centos上构建Ubuntu系统的镜像, <http://singularity.lbl.gov/building-ubuntu-rhel-host>