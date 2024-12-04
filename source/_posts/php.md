之前发布过PHP版切片视频并将视频上传到图床的教程，现在这个是在网页上实现切片视频并上传到图床。

这个项目已失效，仅作为备份

## 序言

我之前的教程在这：  

PHP版切片视频并将其上传到图床其实很简陋，只能解决有无，基本上每用一次就得把机器删掉，也没法自动修复上传失败的地方。  
而在网页上实现可视化切片视频并将其上传到图床不仅实现了可视化，还可以半自动修复上传失败的文件。

如果用宝塔的python项目管理器会简单点，但必须要用centos系统，本文是针对不喜欢用宝塔/在用宝塔但不是centos的用户。

如果你觉得看这篇教程感觉晕乎乎的，那么我建议你看这篇教程。

## 搭建教程

### 安装WEB环境

无论是宝塔还是appnode还是单独Nginx都可以，只是反代的作用，因为我用appnode作为我的探针，因此在这我选择appnode。

### 安装ffmpeg

我们在这依然使用静态预构建版本安装FFMPEG，虽然教程说是centos7，但实测这个在centos和debian上都能用。如果你是centos，建议不要使用yum安装，yum安装的ffmpeg容易出问题。  
获取安装程序脚本：

```
wget https://raw.githubusercontent.com/q3aql/ffmpeg-install/master/ffmpeg-install
```

使它可执行

```
chmod a+x ffmpeg-install
```

安装发行版本

```
./ffmpeg-install --install release
```

确保其有效：

```
root@debian:~# ffmpeg -version
ffmpeg version 4.2.2-static https://johnvansickle.com/ffmpeg/  Copyright (c) 2000-2019 the FFmpeg developers
built with gcc 8 (Debian 8.3.0-6)
configuration: --enable-gpl --enable-version3 --enable-static --disable-debug --disable-ffplay --disable-indev=sndio --disable-outdev=sndio --cc=gcc --enable-fontconfig --enable-frei0r --enable-gnutls --enable-gmp --enable-libgme --enable-gray --enable-libaom --enable-libfribidi --enable-libass --enable-libvmaf --enable-libfreetype --enable-libmp3lame --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-librubberband --enable-libsoxr --enable-libspeex --enable-libsrt --enable-libvorbis --enable-libopus --enable-libtheora --enable-libvidstab --enable-libvo-amrwbenc --enable-libvpx --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxml2 --enable-libdav1d --enable-libxvid --enable-libzvbi --enable-libzimg
libavutil      56. 31.100 / 56. 31.100
libavcodec     58. 54.100 / 58. 54.100
libavformat    58. 29.100 / 58. 29.100
libavdevice    58.  8.100 / 58.  8.100
libavfilter     7. 57.100 /  7. 57.100
libswscale      5.  5.100 /  5.  5.100
libswresample   3.  5.100 /  3.  5.100
libpostproc    55.  5.100 / 55.  5.100
```

### 安装python3

系统默认安装的是python2,在这里我们需要安装python3以及pip3

#### centos

centos只需要

```
yum install epel-release
yum install python36
yum install python36-pip
```

即可获得python3和pip3

#### debian

官方源没python3  
我们编译安装：

```
apt-get update
apt-get install aptitude
aptitude -y install gcc make zlib1g-dev libffi-dev libssl-dev
wget https://www.python.org/ftp/python/3.6.10/Python-3.6.10.tgz
tar -xvf Python-3.6.10.tgz
chmod -R +x Python-3.6.10
cd Python-3.6.10
./configure
make && make install
```

执行python3，pip3检查是否安装完成，不用将python3软链接到python上，进入python3可以使用 ctrl+D 退出。

### 安装git

```
yum install git
```

或者是

```
apt-get install git
```

### 下载源码

```
cd /home
git clone https://github.com/dforel/autoSplitUploadAliImg.git
```

### 安装切片环境

```
pip3 install flask
pip3 install flask_wtf
pip3 install requests
```

### 配置项目管理器

之所以用python项目管理器（uwsgi），~~不仅是作者这样写了这样的入口，~~还因为使用项目管理器可以很轻松的控制python项目占用系统的资源，以及配置开机启动，减轻对其他项目的影响。并且uwsgi.ini内的参数会覆盖掉main.py里设置的参数

宝塔的python项目管理器实际上是用到了UWSGI，UWSGI也是一个项目管理器。  
我们先安装他  
centos:

```
yum install gcc
yum install python-devel
yum install python36-devel
```

debian:  
我安装宝塔的时候应该是自动安装了，貌似没提示要安啥东西

安装libpcre 开发库

debian:

```
apt-get install libpcre3 libpcre3-dev
```

centos:

```
yum install pcre pcre-devel
```

安装uwsgi

```
pip3 install uwsgi
```

修改uwsgi.ini配置文件

```
cd /home/autoSplitUploadAliImg
nano uwsgi.ini
```

uwsgi.ini文件内容如下所示

```
[uwsgi]
master = true
http=192.168.200.10:5000
chdir = /www/wwwroot/abc.com/
wsgi-file=/www/wwwroot/abc.com/main.py
callable=app
processes=1
threads=1
buffer-size = 65536
vacuum=true
pidfile =/www/wwwroot/abc.com/uwsgi.pid
```

```
[uwsgi]
master = true
#启动主进程，来管理其他进程，其它的uwsgi进程都是这个master进程的子进程，如果kill这个master进程，相当于重启所有的uwsgi进程。
http=192.168.200.10:5000
#指定网页监听ip和端口
chdir = /www/wwwroot/abc.com/
#在app加载前切换到当前目录， 指定运行目录
wsgi-file=/www/wwwroot/abc.com/main.py
#运行的入口文件
callable=app
#将功能指定为app
processes=1
#使用一个进程
threads=1
#使用一个线程
buffer-size = 65536
#设置用于uwsgi包解析的内部缓存区大小。默认是4096(4k)，如果http请求数据超过这个量，就会报错。
vacuum=true
#当服务器退出的时候自动删除unix socket文件和pid文件。
pidfile =/www/wwwroot/abc.com/uwsgi.pid
#指定pid文件
```

根据配置文件的含义，手动修改下。

```
[uwsgi]
master = true
http=0.0.0.0:8080
chdir = /home/autoSplitUploadAliImg/
wsgi-file = main.py
callable = app
processes = 1
threads = 1
buffer-size = 32768
vacuum = true
pidfile = uwsgi.pid
```

修改完后保存，并退出。

### 测试

先去宝塔/appnode那先把端口打开，执行

```
python3 main.py
```

正常情况下会提示：

```
 * Serving Flask app "main" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: ***-***-***
```

如果用户访问会出现访问的日志。  
访问地址为`服务器ip:8080`  
例如`8.8.8.8:8080`  
添加个视频，切片测试下，在线观看或者下载切片试试  

由于Windows端不支持特殊字符，下载的m3u8文件名会变成`downTemp`，请自行修改文件名并添加后缀，例如xxx.m3u8  
如需在标签页显示网站图标，请将favicon.ico文件放入autoSplitUploadAliImg/static/文件夹中，默认没有

测试可用后用ctrl+c取消运行。

### 正式部署

先使用root用户测试下能不能正常运行

```
uwsgi uwsgi.ini
```

能进入网页，正常切片，则开始正式部署。  
修改uwsgi.ini如下所示。

```
[uwsgi]
master = true
http=0.0.0.0:8080
chdir = /home/autoSplitUploadAliImg/
wsgi-file = main.py
callable = app
processes = 1
threads = 1
buffer-size = 32768
vacuum = true
pidfile = uwsgi.pid
harakiri = 3600
```

设置一个harakiri计时器，它将破坏停留时间超过指定秒数的进程，我这里选择的是3600秒，即1小时，我不信你有耐心等他处理一小时。

再次测试下能不能正常工作

```
uwsgi uwsgi.ini
```

最好是把所有日志文件复制一下，看看有没有error

### 开机启动

编辑启动文件

```
nano /etc/systemd/system/m3u8.uwsgi.service
```

根据自己的需求修改

```
[Unit]
Description=uWSGI m3u8
After=syslog.target

[Service]
ExecStart=/usr/local/bin/uwsgi  --ini /home/autoSplitUploadAliImg/uwsgi.ini
Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

运行测试

```
chmod +x /etc/systemd/system/m3u8.uwsgi.service
systemctl start m3u8.uwsgi.service
```

检查

```
systemctl status m3u8.uwsgi.service
```

添加开机启动项

```
systemctl enable m3u8.uwsgi.service
```

重启服务器，测试网站。

### 添加域名

上面如果能正常访问，就在宝塔那添加网站，增加https，反代的后端请使用127.0.0.1:8080,并关闭防火墙的8080端口。  

记得修改Nginx限制上传文件大小，不然会报"413 Request Entity Too Large"

### 优化

#### 自动删除临时文件

由于原作者没写，不能自动删除生成的临时文件，以及重启服务器后上传文件夹的文件也没删，因此我就在这增加一个删除脚本,可以自动删除前天创建的临时文件夹，你丢在appnode/宝塔的自动运行脚本功能里就行了。

```
find /home/autoSplitUploadAliImg/temp/*  -mtime +1 -exec rm -rf {} \;
/home/autoSplitUploadAliImg/upload/* --设置查找的目录,不带*会删掉该目录；
-mtime +1 --设置修改时间为1天前；
-exec rm -rf --查找完毕后执行删除操作；
 {} \; --固定写法
```

> find 路径 -mtime +天数 -exec rm -rf {} ;

如果没面板的话可以保存为xxx.sh脚本，用crontab来执行。

#### 自动删除上传文件夹

由于服务重启后，面板那上传文件记录会消失，因此需要自动删除上传文件夹。  
在rc.local内，加入：

```
rm -rf /home/autoSplitUploadAliImg/upload/*
```

即可在重启服务器后，自动删除上传文件夹内的文件。  

不建议对接CDN，cdn限制了上传文件的大小。所以记得设密码，最好不要共享使用。

## 使用说明

### 上传

[![1-upload-mp4.png](https://pic.wnark.com/images/2020/02/12/1-upload-mp4.png "1-upload-mp4.png")](https://pic.wnark.com/images/2020/02/12/1-upload-mp4.png)

[1-upload-mp4.png](https://pic.wnark.com/images/2020/02/12/1-upload-mp4.png)

### 文件检查和分片

[![2-check-cut.png](https://pic.wnark.com/images/2020/02/12/2-check-cut.png "2-check-cut.png")](https://pic.wnark.com/images/2020/02/12/2-check-cut.png)

[2-check-cut.png](https://pic.wnark.com/images/2020/02/12/2-check-cut.png)

分完片之后最好去临时文件夹看看有没有片段大于5M，如果有的话，建议原视频用小丸工具箱处理后再上传切片，小丸工具箱用默认设置即可，直接压制。

### 开始上传分片

[![3-upload-cut.png](https://pic.wnark.com/images/2020/02/12/3-upload-cut.png "3-upload-cut.png")](https://pic.wnark.com/images/2020/02/12/3-upload-cut.png)

[3-upload-cut.png](https://pic.wnark.com/images/2020/02/12/3-upload-cut.png)

### 处理可能失败的分片

[![4-repeat-upload.png](https://pic.wnark.com/images/2020/02/12/4-repeat-upload.png "4-repeat-upload.png")](https://pic.wnark.com/images/2020/02/12/4-repeat-upload.png)

[4-repeat-upload.png](https://pic.wnark.com/images/2020/02/12/4-repeat-upload.png)

### 完成

可下载或者在线播放  
[![5-end.png](https://pic.wnark.com/images/2020/02/12/5-end.png "5-end.png")](https://pic.wnark.com/images/2020/02/12/5-end.png)

[5-end.png](https://pic.wnark.com/images/2020/02/12/5-end.png)

由于Windows端不支持特殊字符，下载的m3u8文件名会变成`downTemp`，请自行修改文件名并添加后缀，例如xxx.m3u8

## 演示

武汉加油（切片上传时间：2020/2/13）  

\[x\]

Player version

Player FPS

Video type

Video url

Video resolution

Video duration

---

参考：  
[FFMPEG INSTALL ON CENTOS 7](https://www.wnark.com/go/aHR0cHM6Ly9saW51eGFkbWluLmlvL2luc3RhbGwtZmZtcGVnLW9uLWNlbnRvcy03Lw==)  
[dforel/autoSplitUploadAliImg](https://www.wnark.com/go/aHR0cHM6Ly9naXRodWIuY29tL2Rmb3JlbC9hdXRvU3BsaXRVcGxvYWRBbGlJbWc=)  
[docker搭建视频切割上传阿里图床的可视化工具](https://www.wnark.com/go/aHR0cHM6Ly94aHZwcy5pbmZvL2luZGV4LnBocC9hcmNoaXZlcy84MS5odG1s)  
[Quickstart for Python/WSGI applications](https://www.wnark.com/go/aHR0cHM6Ly91d3NnaS1kb2NzLnJlYWR0aGVkb2NzLmlvL2VuL2xhdGVzdC9XU0dJcXVpY2tzdGFydC5odG1s)  
[uwsgi参数详解](https://www.wnark.com/go/aHR0cHM6Ly93d3cuY25ibG9ncy5jb20vdG9ydG9pc2U1MTIvcC8xMDgyNTA3NS5odG1s)  
[uwsgi 配置参数](https://www.wnark.com/go/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2NTE0NTg4L2FydGljbGUvZGV0YWlscy84NDU4NzAzNA==)  
[uwsgi默认buffersize太小可能导致问题](https://www.wnark.com/go/aHR0cHM6Ly9jbG91ZC50ZW5jZW50LmNvbS9kZXZlbG9wZXIvYXJ0aWNsZS8xNTExMDQ5)  
[Systemd](https://www.wnark.com/go/aHR0cHM6Ly91d3NnaS1kb2NzLnJlYWR0aGVkb2NzLmlvL2VuL2xhdGVzdC9TeXN0ZW1kLmh0bWw=)  
[linux定时删除N天前的文件（文件夹）](https://www.wnark.com/go/aHR0cHM6Ly93d3cuamlhbnNodS5jb20vcC9iN2FlODhkMGEwMWU=)

---

> 版权属于：[寒夜方舟](https://www.wnark.com/)
> 
> 本文链接：[https://www.wnark.com/archives/58.html](https://www.wnark.com/archives/58.html)
> 
> 本站所有原创文章采用[署名-非商业性使用 4.0 国际 (CC BY-NC 4.0)](https://www.wnark.com/go/aHR0cHM6Ly9jcmVhdGl2ZWNvbW1vbnMub3JnL2xpY2Vuc2VzL2J5LW5jLzQuMC9kZWVkLnpo)。 您可以自由地转载和修改，但请注明引用文章来源和不可用于商业目的。声明：本博客完全禁止任何商业类网站转载，包括但不限于CSDN，51CTO，百度文库，360DOC，AcFun，哔哩哔哩等网站。