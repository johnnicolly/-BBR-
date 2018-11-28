# -BBR-
本脚本适用环境 系统支持：CentOS 6+，Debian 7+，Ubuntu 12+ 虚拟技术：OpenVZ 以外的，比如 KVM、Xen、VMware 等 内存要求：≥128M 日期　　：2018 年 06 月 09 日
关于本脚本
1、本脚本已在 Vultr 上的 VPS 全部测试通过。
2、当脚本检测到 VPS 的虚拟方式为 OpenVZ 时，会提示错误，并自动退出安装。
3、脚本运行完重启发现开不了机的，打开 VPS 后台控制面板的 VNC, 开机卡在 grub 引导, 手动选择内核即可。
4、由于是使用最新版系统内核，最好请勿在生产环境安装，以免产生不可预测之后果。

使用方法
使用root用户登录，运行以下命令：

wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启。
重启完成后，进入 VPS，验证一下是否成功安装最新内核并开启 TCP BBR，输入以下命令：

uname -r
查看内核版本，显示为最新版就表示 OK 了

sysctl net.ipv4.tcp_available_congestion_control
返回值一般为：
net.ipv4.tcp_available_congestion_control = bbr cubic reno
或者为：
net.ipv4.tcp_available_congestion_control = reno cubic bbr

sysctl net.ipv4.tcp_congestion_control
返回值一般为：
net.ipv4.tcp_congestion_control = bbr

sysctl net.core.default_qdisc
返回值一般为：
net.core.default_qdisc = fq

lsmod | grep bbr
返回值有 tcp_bbr 模块即说明 bbr 已启动。注意：并不是所有的 VPS 都会有此返回值，若没有也属正常。

CentOS 下最新版内核 headers 安装方法
本来打算在脚本里直接安装 kernel-ml-headers，但会出现和原版内核 headers 冲突的问题。因此在这里添加一个脚本执行完后，手动安装最新版内核 headers 之教程。
执行以下命令

yum --enablerepo=elrepo-kernel -y install kernel-ml-headers
根据 CentOS 版本的不同，此时一般会出现类似于以下的错误提示：

Error: kernel-ml-headers conflicts with kernel-headers-2.6.32-696.20.1.el6.x86_64
Error: kernel-ml-headers conflicts with kernel-headers-3.10.0-693.17.1.el7.x86_64
因此需要先卸载原版内核 headers ，然后再安装最新版内核 headers。执行命令：

yum remove kernel-headers
确认无误后，输入 y，回车开始卸载。注意，有时候这么操作还会卸载一些对内核 headers 依赖的安装包，比如 gcc、gcc-c++ 之类的。不过不要紧，我们可以在安装完最新版内核 headers 后再重新安装回来即可。
卸载完成后，再次执行上面给出的安装命令。

yum --enablerepo=elrepo-kernel -y install kernel-ml-headers
成功安装后，再把那些之前对内核 headers 依赖的安装包，比如 gcc、gcc-c++ 之类的再安装一次即可。

为什么要安装最新版内核 headers 呢？
这是因为 shadowsocks-libev 版有个 tcp fast open 功能，如果不安装的话，这个功能是无法开启的。

内核升级方法
如果是 CentOS 系统，执行如下命令即可升级内核：

yum -y install kernel-ml kernel-ml-devel
如果你还手动安装了新版内核 headers ，那么还需要以下命令来升级 headers ：

yum -y install kernel-ml-headers
CentOS 6 的话，执行命令：

sed -i 's/^default=.*/default=0/g' /boot/grub/grub.conf
CentOS 7 的话，执行命令：

grub2-set-default 0
如果是 Debian/Ubuntu 系统，则需要手动下载最新版内核来安装升级。
去这里下载最新版的内核 deb 安装包。
如果系统是 64 位，则下载 amd64 的 linux-image 中含有 generic 这个 deb 包；
如果系统是 32 位，则下载 i386 的 linux-image 中含有 generic 这个 deb 包；
安装的命令如下（以最新版的 64 位 4.12.4 举例而已，请替换为下载好的 deb 包）：

dpkg -i linux-image-4.12.4-041204-generic_4.12.4-041204.201707271932_amd64.deb
安装完成后，再执行命令：

/usr/sbin/update-grub
最后，重启 VPS 即可。

特别说明
如果你使用的是 Google Cloud Platform （GCP）更换内核，有时会遇到重启后，整个磁盘变为只读的情况。只需执行以下命令即可恢复：

mount -o remount rw /
轉自https://teddysun.com/489.html，感谢
