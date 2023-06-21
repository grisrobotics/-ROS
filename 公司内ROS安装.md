公司内ROS安装注意：

添加中科大的源  X86_64 架构

```
mv /etc/apt/sources.list /etc/apt/sources.list.back
sudo vim /etc/apt/sources.list
```

```
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

添加ROS镜像源：

```
sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.ustc.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'
```

由于使用的是公司内网所以需要将**http**改成**https**

```
sudo sh -c '. /etc/lsb-release && echo "deb https://mirrors.ustc.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'
```

然后更新源

```
sudu apt-get update
```

随后安装melodic

```
sudo apt install ros-melodic-desktop-full
```

ROS初始化，配置全局环境变量（源）：

```
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

安装打包工具依赖：

```
sudo apt install python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential
```





**初始化 rosdep**

```
sudo rosdep init
rosdep update
```

如果此时第二天命令报错：

```
ERROR: default sources list file already exists:
/etc/ros/rosdep/sources.list.d/20-default.list
Please delete if you wish to re-initialize
```

则需要删除初始化文件

```
sudo rm /etc/ros/rosdep/sources.list.d/20-default.list
```

若是删除之后还是报错

```
ERROR: cannot create /etc/ros/rosdep/sources.list.d/20-default.list:
[Errno 13] Permission denied: '/etc/ros/rosdep/sources.list.d/20-default.list'
```

修改`/etc/resolv.conf` 文件

```
sudo gedit /etc/resolv.conf
```

```
将原有的nameserver那一行注释，并添加以下两行后保存：
  nameserver 8.8.8.8 #google域名服务器
  nameserver 8.8.4.4 #google域名服务器
```

修改保存后在执行rosdep init成功





rosdep update失败：

```
sudo gedit /usr/lib/python2.7/dist-packages/rosdep2/sources_list.py
```

```
def download_rosdep_data(url):
    """
    :raises: :exc:`DownloadFailure` If data cannot be
        retrieved (e.g. 404, bad YAML format, server down).
    """
    try:
		url = "https://ghproxy.com/" + url
        f = urlopen_gzip(url, timeout=DOWNLOAD_TIMEOUT)
        text = f.read()
        f.close()
        data = yaml.safe_load(text)
        if type(data) != dict:
            raise DownloadFailure('rosdep data from [%s] is not a YAML dictionary' % (url))
        return data
    except (URLError, httplib.HTTPException) as e:
        raise DownloadFailure(str(e) + ' (%s)' % url)
    except yaml.YAMLError as e:
        raise DownloadFailure(str(e))
```

在这个函数的try下面加上下面这一行，注意对齐，加完后如上所示

```
url = "https://ghproxy.com/" + url
```



修改如下文件

```
sudo gedit /usr/lib/python2.7/dist-packages/rosdistro/__init__.py
```

在 `https://raw.githubusercontent.........` 前加入 `https://ghproxy.com/`





以同样的方式在以下文件添加 `https://ghproxy.com/`

sudo gedit /usr/lib/python2.7/dist-packages/rosdep2/gbpdistro_support.py	第35行
sudo gedit /usr/lib/python2.7/dist-packages/rosdep2/sources_list.py	第69行
sudo gedit /usr/lib/python2.7/dist-packages/rosdep2/rep3.py	第39行
sudo gedit /usr/lib/python2.7/dist-packages/rosdistro/manifest_provider/github.py	第68行、第119行

具体行数可能会改变，按照实际情况对URL链接进行修改

修改文件 sudo gedit /usr/lib/python2.7/dist-packages/rosdep2/gbpdistro_support.py 

在206行添加

```
gbpdistro_url = "https://ghproxy.com/" + gbpdistro_url
```



总共需要修改7个文件

可能会出现如下图所示的问题

![image-20230619112535345](C:\Users\1429882\AppData\Roaming\Typora\typora-user-images\image-20230619112535345.png)

多试几次就可以成功了



![image-20230619104916399](C:\Users\1429882\AppData\Roaming\Typora\typora-user-images\image-20230619104916399.png)