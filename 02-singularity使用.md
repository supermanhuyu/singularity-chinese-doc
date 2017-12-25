---
title: singularity使用手册
---

# 制作容器

## 选项一 导入docker容器

使用现成的docker容器，比如docker hub上面的tensorflow：

```
sudo singularity create --size 4000 tensorflow.img
sudo singularity import tensorflow.img docker://tensorflow/tensorflow:latest
```

## 选项二 自己制作singularity容器

制作一个singularity容器需要两步，需要在任何一台有root权限的linux计算机上进行：

1 创建空白镜像文件

`sudo singularity create --size 1200  keras2.0_cpu.img`

其中 size用来制定大小，单位是MB，上面将创建一个1.2GB的空白镜像文件。

2 接下来需要在这个镜像中写入系统和预装的软件

创建 keras2.0_cpu.def 文件，内容通过符号%分割成几个部分，分别代表不同的功能。

我们主要要写的内容是1） %setup；  2）%post

```
# 基于docker的ubuntu镜像
BootStrap: docker
From: ubuntu:16.04

%runscript
    echo "This is what happens when you run the container..."
    bash
%setup
    # 1）这一步可以先设置好环境变量，通过wget等下载好必要的软件库等，注意这些指令都只是在host机器上运行，不是container里面。
    # 内容请参考https://git.oschina.net/sg-ai/singularityimages/blob/master/keras-tf-1.0.1-gpu.setup
%post
    # non interactive debian
    DEBIAN_FRONTEND=noninteractive
    # Install the necessary packages (from repo)
    apt-get update && apt-get install -y --no-install-recommends curl ca-certificates

    # 2）这些指令都在container里面运行，在这里写你要安装和设置的东西，可以使用apt install之类的安装需要的软件，这样就会被安装到容器里面去
    # 内容请参考https://git.oschina.net/sg-ai/singularityimages/blob/master/keras-tf-1.0.1-gpu.post

```

接着运行：

```
sudo singularity bootstrap keras2.0_cpu.img keras2.0_cpu.def

```

这样keras2.0_cpu.img 这个文件就可以用了，把它拷贝到集群上，就可以通过singularity使用。

# 使用singularity容器

## 运行容器环境

singularity容器中提供隔离的软件环境，方便各种测试、开发、甚至产品发布，同时它专门为集群设计，因为可以不需要root权限即可运行。

常用的singularity指令有2中shell和exec, 比如，以下指令可以打开一个互动的shell：

```
singularity shell /atlas/containers/keras2.0_cpu.img
```

你会看到：

```
[wouyang@login images_singularity]$ singularity shell ./keras2.0_cpu.img

Singularity: Invoking an interactive shell within container...

Singularity.keras-2.0.3-py3-tf-gpu.img> 
```

接下来你就可以在这个容器里面运行任何指令了， 但是注意这个容器是只读的，你不能安装新的东西。

如果你不是想敲单条指令，你可以直接使用exec指令来运行你的python文件，例如

```
singularity exec ./keras2.0_cpu.img python3 hello.py

```

## 添加新软件/修改容器

使用过程中，如果需要安装新的软件，可以使用--writable 或者 -w 参数来对容器进行修改，运行以下指令：

```
sudo singularity shell --writable ./keras2.0_cpu.img

```

接下来打开的shell里面，可以使用apt-get、pip等安装新的软件，修改会直接写入到容器中。

## singularity帮助使用

直接输入singularity即可查看帮助

```
[yhu@node7 ~]$singularity 
USAGE: singularity [global options...] <command> [command options...] ...

GLOBAL OPTIONS:
    -d --debug    Print debugging information
    -h --help     Display usage summary
    -q --quiet    Only print errors
       --version  Show application version
    -v --verbose  Increase verbosity +1
    -x --sh-debug Print shell wrapper debugging information

GENERAL COMMANDS:
    help          Show additional help for a command

CONTAINER USAGE COMMANDS:
    exec          Execute a command within container
    run           Launch a runscript within container
    shell         Run a Bourne shell within container
    test          Execute any test code defined within container

CONTAINER MANAGEMENT COMMANDS (requires root):
    bootstrap     Bootstrap a new Singularity image from scratch
    copy          Copy files from your host into the container
    create        Create a new container image
    expand        Grow the container image
    export        Export the contents of a container via a tar pipe
    import        Import/add container contents via a tar pipe
    mount         Mount a Singularity container image

```

如果要看command的具体使用,可使用如下命令,例如要查看 singularity shell 如何使用

```
[yhu@node7 ~]$singularity help shell 
USAGE: singularity [...] shell [shell options...] <container path>

Obtain a shell (/bin/sh) within the container image.

note: When invoking a shell within a container, the container image is
by default writable.


SHELL OPTIONS:
    -B/--bind <spec>    A user-bind path specification.  spec can either be a
                        path or a src:dest pair, specifying the bind mount to
                        perform (can be called multiple times)
    -c/--contain        This option disables the automatic sharing of writable
                        filesystems on your host (e.g. $HOME and /tmp).
    -C/--containall     Contain not only file systems, but also PID and IPC
    -H/--home <path>    Path to a different home directory to virtualize within
                        the container
    -i/--ipc            Run container in a new IPC namespace
    -p/--pid            Run container in a new PID namespace (creates child)
    --pwd               Initial working directory for payload process inside
                        the container
    -S/--scratch <path> Include a scratch directory within the container that
                        is linked to a temporary dir (use -W to force location)
    -s/--shell <shell>  Path to program to use for interactive shell
    -u/--user           Try to run completely unprivileged (only works on very
                        new kernels/distros)
    -W/--workdir        Working directory to be used for /tmp, /var/tmp and
                        $HOME (if -c/--contain was also used)
    -w/--writable       By default all Singularity containers are available as
                        read only. This option makes the file system accessible
                        as read/write.


NOTE:
    If there is a daemon process running inside the container, then 
    subsequent container commands will all run within the same namespaces.
    This means that the --writable and --contain options will not be
    honored as the namespaces have already been configured by the
    'singularity start' command.


EXAMPLES:

    $ singularity shell /tmp/Debian.img
    Singularity/Debian.img> pwd
    /home/gmk/test
    Singularity/Debian.img> exit
    $ singularity shell -C /tmp/Debian.img
    Singularity/Debian.img> pwd
    /home/gmk
    Singularity/Debian.img> ls -l
    total 0
    Singularity/Debian.img> exit
    $ sudo singularity shell -w /tmp/Debian.img

```

注意一点,singularity进入容器后,会同服务器共享一个家目录.那如果要在容器中运行项目上的程序,而程序又不在你的家目录比如在/atlas/project/MD 该怎么办呢?

可以这样,加一个参数 -H 接上路径,这个路径就会共享了.例:

```
singularity shell -H /atlas/projects/MD /atlas/containers/k-n-g.img

```

增:后来发现同样也可用 -B参数来实现

```
singularity shell -B /atlas/projects/MD:/mnt  /atlas/containers/k-n-g.img

```

这个命令意思是,当你进入容器的/mnt目录的时候,看到的是 /atlas/projects/MD 中的内容

# 在集群上使用slurm和singularity

在集群上，可以通过slurm系统直接将singularity发送到计算节点上，只需要在原先的指令前加slurm相关指令和参数即可，例如：

```
srun -p common singularity exec ./keras2.0_cpu.img python3 hello.py
```

```
# 指定使用GPU节点
srun --gres=gpu -p fast singularity exec /atlas/containers/keras-2.0.3-py3-tf-gpu.img python3 mnist_cnn.py

```

## 运行互动脚本环境--pty选项

运行可以交互的shell脚本，一般的srun指令下，是不支持脚本互动的，如果要在某个节点上使用shell、bash、ipython等互动脚本环境，需要加上--pty选项

例如

```
# 在集群计算节点上执行python互动
[wouyang@login ~]$ srun --pty -p common python

Python 2.7.5 (default, Nov 20 2015, 02:00:19) 

[GCC 4.8.5 20150623 (Red Hat 4.8.5-4)] on linux2

Type "help", "copyright", "credits" or "license" for more information.

>>> 
```

同理，如果要使用singularity容器里面的shell，应该加上--pty

```
# 在集群计算节点上执行shell互动
[wouyang@login ~]$ srun --pty -p common singularity shell /atlas/containers/keras-2.0.3-py3-tf-gpu-notebook-nodejs.img
```

## 实例一 集群上使用运行ipython环境

```
# 在login节点上测试，注意，仅供测试，不要做长时间运算
[wouyang@login ~]$ singularity exec /atlas/containers/keras-2.0.3-py3-tf-gpu-notebook-nodejs.img ipython

# 另外，计算量大的需要发送到计算节点上进行，并且要及时关闭，否则会一直占用节点资源
[wouyang@login ~]$ srun --pty -p common singularity exec /atlas/containers/keras-2.0.3-py3-tf-gpu-notebook-nodejs.img ipython
```

## 实例二 在集群上运行jupyter notebook

【注意】不推荐在集群上使用jupyter notebook，因为它会长时间占用节点和GPU资源，因此仅供短时间测试使用。

singularity容器会自动带入当前用户的所有环境变量，但是对于jupyter notebook，目前集群上的$XDG_RUNTIME_DIR 指向/run/user文件夹，这个文件夹在容器里面不存在，所以会导致问题，为了防止singularity自动传入当前环境变量到容器，可以通过-c选项来进行：

```
# 在集群上运行包含jupyter notebook的容器， -c选项防止将当前的环境变量传入到容器
srun -p common singularity exec /atlas/containers/keras-2.0.3-py3-tf-gpu-notebook-nodejs.img -c jupyter notebook --ip=0.0.0.0 --port=8888
```

另外， --ip=0.0.0.0 选项确保在容器外部能访问内部网络。