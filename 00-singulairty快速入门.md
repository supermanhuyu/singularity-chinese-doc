# singularity 快速入门

## 快速安装

安装有多种方式，快速入门就是告诉你一个简单的就行：

```
git clone https://github.com/singularityware/singularity.git
cd singularity
./autogen.sh
./configure --prefix=/usr/local
make
sudo make install #这里你要有sudo权限哦  或者用root直接make install也行
```

## 快速使用

singularity使用超简单，跟docker大同小异，查看帮助是最好最快最有效的方法

```
singularity --help
USAGE: singularity [global options...] <command> [command options...] ...

GLOBAL OPTIONS:
    -d|--debug    Print debugging information
    -h|--help     Display usage summary
    -s|--silent   Only print errors
    -q|--quiet    Suppress all normal output
       --version  Show application version
    -v|--verbose  Increase verbosity +1
    -x|--sh-debug Print shell wrapper debugging information

GENERAL COMMANDS:
    help       Show additional help for a command or container                  
    selftest   Run some self tests for singularity install                      

CONTAINER USAGE COMMANDS:
    exec       Execute a command within container                               
    run        Launch a runscript within container                              
    shell      Run a Bourne shell within container                              
    test       Launch a testscript within container                             

CONTAINER MANAGEMENT COMMANDS:
    apps       List available apps within a container                           
    bootstrap  *Deprecated* use build instead                                   
    build      Build a new Singularity container                                
    check      Perform container lint checks                                    
    inspect    Display a container's metadata                                   
    mount      Mount a Singularity container image                              
    pull       Pull a Singularity/Docker container to $PWD                      

COMMAND GROUPS:
    image      Container image command group                                    
    instance   Persistent instance command group                                


CONTAINER USAGE OPTIONS:
    see singularity help <command>

For any additional help or support visit the Singularity
website: http://singularity.lbl.gov/

```

如果你想进一步了commands的相关信息，帮助文档提供了各种姿势的帮助：

```
$ singularity help <command>
$ singularity --help <command>
$ singularity -h <command>
$ singularity <command> --help
$ singularity <command> -h
```

如果image制作者谢了帮助在里面，也可以通过singularity的帮助查看：

```
$ singularity help container.simg            # See the container's help, if provided
```

甚至里面的一个模块如果有帮助信息的话，也可以查看：

```
$ singularity help --app foo container.simg  # See the help for foo, if provided
```

## 下载构建之前的基础镜像

同样类似于docker，是用pull命令

```
$ singularity pull shub://vsoch/hello-world   # 默认镜像名 vsoch-hello-world-master.simg
$ singularity pull --name hello.simg shub://vsoch/hello-world   # 制定了镜像名字
```

shub就是singularity hub的意思，docker有repository，同样singularity也有，而且singularity还可以pull docker的镜像。

```
$ singularity pull docker://godlovedc/lolcow  # with default name
$ singularity pull --name funny.simg docker://godlovedc/lolcow # with custom name
```

docker build会存在这样一个问题，今天pull的一个image和几个月以前pull的image不一定一样，如果某一层的内容稍有改动， 那整个image就要改动，重头pull。对于这样的问题，singularity build有对这个问题做改善。也可以通过singularity build 制作基础镜像，这里要指定一个image的name，如下所示：

```
$ singularity build hello-world.simg shub://vsoch/hello-world
$ singularity build lolcow.simg docker://godlovedc/lolcow
```

## 使用镜像

###shell命令

直接 singularity shell  `<image-name>` 就运行了一个容器，在里面是一个完整的系统

```
$ singularity shell hello-world.simg
Singularity: Invoking an interactive shell within container...

# I am the same user inside as outside!
Singularity hello-world.simg:~/Desktop> whoami
vanessa

Singularity hello-world.simg:~/Desktop> id
uid=1000(vanessa) gid=1000(vanessa) groups=1000(vanessa),4(adm),24,27,30(tape),46,113,128,999(input)
```

这里shell可以运行的容器可不止simg格式一种，通过帮助可以看到可以支持shub，以及docker

```
$singularity help shell
USAGE: singularity [...] shell [shell options...] <container path>
Obtain an interactive shell (/bin/bash) within the container image.
...
CONTAINER FORMATS SUPPORTED:
    *.sqsh              SquashFS format.  Native to Singularity 2.4+
    *.img               This is the native Singularity image format for all
                        Singularity versions < 2.4.
    *.tar*              Tar archives are exploded to a temporary directory and
                        run within that directory (and cleaned up after). The
                        contents of the archive is a root file system with root
                        being in the current directory. Compression suffixes as
                        '.gz' and '.bz2' are supported.
    directory/          Container directories that contain a valid root file
                        system.
    instance://*        A local running instance of a container. (See the
                        instance command group.)
    shub://*            A container hosted on Singularity Hub
    docker://*          A container hosted on Docker Hub
```

### exec命令

exec 允许你在image后面加入你想要执行的命令，同样支持`shub://` and `docker://`

```
$ singularity exec hello-world.simg ls /
anaconda-post.log  etc	 lib64	     mnt   root  singularity  tmp
bin		   home  lost+found  opt   run	 srv	      usr
dev		   lib	 media	     proc  sbin  sys	      var
```

###run命令

直接运行image中默认的命令，同样支持`shub://` and `docker://`

```
$ singularity run shub://GodloveD/lolcow
```

###目录映射

singularity于docker不同的一个地方就是目录映射，singularity默认会映射 `/home/$USER`, `/tmp`, and `$PWD` ，以及其他的可以自己再加：

```
$ echo "Hello World" > $HOME/hello-kitty.txt
$ singularity exec vsoch-hello-world-master.simg cat $HOME/hello-kitty.txt
Hello World
```

## 从头构建镜像（Build images from scratch）

从singularity v2.4开始，build默认构建出来的image是以simg结尾的（上面的例子就是）不能修改内容的格式 the squashfs file format.。这种格式保证了随处可用环境统一，而且可通过md5等手段验证是指定images而没有被别人篡改。

但在测试或者开发过程中，你可能想要安装一些包，修改一些文件，直到你满意为止。singularity还有两种其他的image格式：sandbox 格式，就是一个可以chroot的目录； img格式：2.4以前的版本一直用这个，可读可写，如果要修改或者安装到img中的时候，加上`--witable` 选项即可。想要知道singularity关于文件系统个是的更多详细内容，以及他们之间的区别看这里：<http://singularity.lbl.gov/docs-flow>

下面简单介绍这两种种格式使用方法

### Sandbox Directory

构建时加一个`--build` 选项即可

```
sudo singularity build --sandbox ubuntu/ docker://ubuntu
```

这样就创建了一个ubuntu 目录，目录中是一个完整的ubuntu系统和一些singularity的元数据，之后你就可以用singularity像使用image一样使用这个目录，包括`shell`，`exec`，`run`等命令，不过你做的修改只对本次有用，当你退出来之后目录中的文件并没有改动，要想修改生效就加一个`--writable`选项

```
singularity shell --writable ubuntu/
```

### Writable Image

img格式文件跟sandbox大同小异，构建保存的时候是一个img格式文件，默认只读，要想修改生效就加一个`--writable`选项

```
sudo singularity build --writable ubuntu.img docker://ubuntu
sudo singularity shell --writable ubuntu.img
```

### 格式转换

从一个格式转换到另外一个格式很简单，比如你要从sandbox转换成simg格式(squashfs)的文件

```
$ singularity build new-squashfs sandbox
```

### Singularity Recipes

就是singularity的dockerfile，通过这个配置文件配置你想要的image，不过有些关键字不太一样，下面是一个例子

```
Bootstrap: shub
From: singularityhub/ubuntu

%runscript
    exec echo "The runscript is the containers default runtime command!"

%files
   /home/vanessa/Desktop/hello-kitty.txt        # copied to root of container
   /home/vanessa/Desktop/party_dinosaur.gif     /opt/the-party-dino.gif #

%environment
    VARIABLE=MEATBALLVALUE
    export VARIABLE

%labels
   AUTHOR vsochat@stanford.edu

%post
    apt-get update && apt-get -y install python3 git wget
    mkdir /data
    echo "The post section is where you can install, and configure your container."
```

构建命令：

```
sudo singularity build ubuntu.simg Singularity
```

上面的命令简单易懂，首先bootstrap是告诉你从singularity hub中获取基础镜像，名称叫`singularityhub/ubuntu` ,`runscript` 运行前执行命令，`files` 要copy到image中的文件，`environment` 环境变量，`labels` 标签，`post` 基础镜像构建完成之后执行的命令。

到这里基本的singularity你就会构建和使用了。

