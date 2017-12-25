首先安装呢，在Linux，mac和Windows都可以，但是Windows和mac上都是要装`Vagrant` 的，我没整过，请移步[官网](http://singularity.lbl.gov/install-mac)。 下面直接介绍Linux系统的安装方式

1、源码安装

源码包 [下载地址](http://singularity.lbl.gov/all-releases) ，然后解压-编译-安装即可。也可按照如下方式下载制定版本，编译安装。

```
VERSION=2.4.2
wget https://github.com/singularityware/singularity/releases/download/$VERSION/singularity-$VERSION.tar.gz
tar xvf singularity-$VERSION.tar.gz
cd singularity-$VERSION
./configure --prefix=/usr/local
make
sudo make install
```

或者你直接使用git从[GitHub](https://github.com/singularityware/singularity)上面,下载最新的

```
git clone https://github.com/singularityware/singularity.git
cd singularity
./autogen.sh
./configure --prefix=/usr/local
make
sudo make install
```

安装完成之后直接输入`singularity`，即可验证是否安装成功

```
[superman@node7 ~]$singularity 
USAGE: singularity [global options...] <command> [command options...] ...

GLOBAL OPTIONS:
    -d|--debug    Print debugging information
    -h|--help     Display usage summary
...
```



2、Debian和Ubuntu安装包

 现在Debian和Ubuntu已经有了安装包，添加源即可apt安装：

```
sudo wget -O- http://neuro.debian.net/lists/xenial.us-ca.full | sudo tee /etc/apt/sources.list.d/neurodebian.sources.list
sudo apt-key adv --recv-keys --keyserver hkp://pool.sks-keyservers.net:80 0xA5D32F012649A5A9
sudo apt-get update
sudo apt-get install -y singularity-container

```

3、搭建集群的话，可以制作好RPM/deb包方便部署

a、RPM包制作：

```
git clone https://github.com/singularityware/singularity.git
cd singularity
./autogen.sh
./configure
make dist
rpmbuild -ta singularity-*.tar.gz
sudo yum install ~/rpmbuild/RPMS/*/singularity-[0-9]*.rpm
```

b、deb包制作：

```
#安装依赖
sudo apt-get install -y build-essential libtool autotools-dev automake autoconf 
fakeroot dpkg-buildpackage -b -us -uc 
sudo dpkg -i ../singularity-container_2.3_amd64.deb
```

