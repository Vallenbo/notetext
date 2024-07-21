

# pip工具使用

curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py  # 下载安装脚本

yum install python3-pip -y	# linux安装pip



python get-pip.py   #运行安装脚本

pip3 list --outdated #pip检查哪些包需要更新

pip3 install 包名(模块) #安装

pip3 install -U pip #升级 pip  或easy_install --upgrade pip命令

pip3 uninstall SomePackage #卸载包

pip3 search SomePackage #搜索包

pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple	#永久安装源，清华的镜像源每五分钟更新一次，大而全

pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple package --trusted-host mirrors.aliyun.com

\#临时使用清华大学开源软件镜像站



## 离线依赖包安装

[Python库 搜索](https://pypi.org/)

pip3 download flask #下载flask依赖库，到本级路径

pip3 install pocketsphinx-0.1.15-cp39-cp39-win_amd64.whl #python3安装本地whl



# 升级Python 编译安装

[Python Release Python 3.12.1 下载](https://www.python.org/downloads/release/python-3121/)

1、解压源码包

```sh
root@master ~# tar -xf Python-3.12.0.tar.xz
```

2、编译安装

```sh
root@master ~# ./configure
root@master ~# make
root@master ~# make altinstall
```

注意：

​	这将安装新的Python版本并将其设置为系统的默认Python。如果你不想更改系统的默认Python，你可以在配置步骤中使用`--prefix`选项指定一个安装位置，然后将该位置添加到你的PATH环境变量中。

```bash
./configure --prefix=/path/to/install
make
sudo make install
export PATH=/path/to/install/bin:$PATH
```

这样，你就可以通过指定的路径访问新安装的Python版本，而不会影响系统的默认Python。



# python修改系统默认版本命令

要修改 `pip` 默认使用的 Python 版本，可以使用以下命令：

```sh
$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.12 1
$ sudo update-alternatives --install /usr/bin/pip pip /usr/local/bin/pip3.12 1
```

以上命令将 `/usr/bin/python` 和 `/usr/bin/pip` 两个命令的默认路径分别指向 Python 3.12 版本和对应版本的 `pip` 命令。

执行完上述命令后，你可以通过以下命令验证是否已成功修改 `pip` 的默认 Python 版本：

```sh
$ pip -V
```

如果输出的版本号与你选择的 Python 版本相同，则说明已成功修改默认版本。