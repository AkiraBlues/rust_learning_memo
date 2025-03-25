### WMWare相关问题记录

因为工作需要需要在WINDOWS系统下安装WMWARE以安装UBUNTU，遇到了很多问题，这里统一记录一下。

这里选择WMware wortstation pro 17，Ubuntu 22版本，参考其他教程安装。

安装完成后就当我有了一个Ubuntu电脑。



#### 虚拟机相关安装和配置

登录VMware官网，注册账号，然后获取workstation pro 17下载地址。

之后去下载一个Ubuntu的镜像，大概4G多，可以选清华的那个。

然后安装WMware，然后安装Ubuntu镜像，最后进入系统。

然后是非常麻烦的网络配置，要求是虚拟机可以使用宿主的代理，测试了多个方案，最后的结果是这样：

- clash verge取消tun模式，回到原始的系统代理模式（后面测试恢复到tun模式又可以了，配置按照系统代理模式的走就可以，tun代理适配器不用做任何共享）
- 虚拟机关机情况下，网络适配器设置为NAT，或者自定义然后选VMnet8，这个8就是适配NAT的，NAT本身可以理解为宿主和虚拟机共享的一套网络（也就是宿主也要处于这个网络的管理之下）
- 然后是进入VMware的虚拟网络编辑器，新建一个VMnet1，对应仅主机模式，然后新建一个VMnet8，对应NAT模式，其他IP配置的先不要动，然后点击左下角还原默认设置，一段时间后，去控制面板网络适配器里面看，会发现出现了net1和net8的适配器，不需要在LAN或者WLAN的适配器上配置共享
- 还原默认设置后，会发现IP地址发生变化了，重点记录net8的IP地址，可以在任务管理器的进程里面确认，通常是192.168.X.X，它会和宿主的网段区隔开，比如宿主的网段是192.168.31.X，对应家里的路由器网段，那么它这个肯定不会是31
- 然后配置clash的代理端口，允许局域网访问，然后开虚拟机进入Ubuntu系统，进入系统配置，把这个net8的IP地址输入进去，这个就是虚拟机通过net8网络找到宿主的IP地址，也就相当于是宿主开热点然后另外一台机器连接了，此时宿主的热点就是一个net8网络，里面也给宿主分配了一个新的IP地址
- 用火狐浏览器测试外网是否能访问，如果可以则成功，也可以从clash的日志里面看到虚拟机的访问情况
- 还要在终端里面去配置一下代理，系统设置的代理对终端的很多CURL命令无效，之后配置好后可以直接通过curl去安装rustup的linux版本了

然后是在linux环境下操作了，先直接安装DOCKER，因为这个对系统影响最大，另外不要在/etc/apt/souces.list里面去加清华的源，会导致包版本不一致，导致后续安装KVM虚拟化相关失败，另外建议最好搞一个磁盘清理的，比如bleachbit，时刻掌控磁盘空间大小，避免SDA3分区盛满的情况。

安装rustup，然后是VS CODE，之后是GIT，其他相关所需的，安装cargo-stylus时，先安装pkg-config，然后安装libssl-dev，再安装cargo-stylus，最后终于构建成功，有了CARGO-STYLUS工具链，之后再引入一个DEMO应用，然后编译构建，终于成功了。

之后流程参考Arbitrum官方的教程，流程是这样的：

- RUST开发环境，即RUST相关工具，环境，和IDE，这部分之前已经提到
- Docker，这部分后面会介绍
- Foundry CLI，用于和EVM合约进行交互
- Nitro开发节点，需要从GITHUB上下载一个开发节点项目并启动它，它的脚本会触发docker相关行为



#### Docker相关

Docker介绍，它是一个公开的平台，可以用来开发应用。Docker可以在一个沙箱中构建和运行应用，甚至在多个沙箱中做同样的事情。沙箱非常轻量，它包含了运行应用所需的所有环境，因此不需要直接在宿主环境（一般指个人用开发电脑）上安装对应的开发环境。一个最简单的过程就是通过Docker搭建一个CI / CD环境，这样每次代码提交后都可以直接在Docker上进行自动代码审核，打包构建，测试，发布等一系列流程，而不用去在自己的电脑上直接搭建这样的环境。

Docker安装过程记录：

- 确认OS，linux下需要开启KVM虚拟化，由于我是通过虚拟机跑的Ubuntu，因此实际上是通过虚拟机的虚拟化去支持KVM虚拟化，宿主Windows11环境本身需要调整一下，然后是虚拟机开启对应虚拟化配置，然后才是Ubuntu自身安装一些软件去开启KVM虚拟化，并授权给当前用户
- 之后参考Docker官方教程去安装，还是通过下载deb包的形式安装
- 然后在设置里面的docker engine修改一下国内镜像源，写法是`"registry-mirrors": ["https://some-domain.com",
  "https://some-domain2.com"]`，这里要改成实际的镜像源
- resource里面改一下memory limit，最少要4G

然后就开始涉及DOCKER最重要的2个概念，container（容器）和image（镜像），首先要下载镜像，就是操作OS，它是静态的，不可修改的，然后基于这个静态的镜像可以创建多个运行实例，就是容器，每个容器都是动态的，可以修改的，而且各自独立。

DOCKER安装好之后可以在gitbash或者CMD使用`docker --help`来确认它安装好了，并且可以查询各种命令。WMware内安装Docker会有点兼容性问题，关闭之后无法再次打开，此时进入任务管理器杀一下docker.backend进程就可以了。

之后在DOCKER的主界面可以打开终端，然后执行以下命令，会开始下载镜像并创建容器：`docker run -d -p 8080:80 docker/welcome-to-docker`，会先尝试加载本地镜像，失败后从镜像源下载对应镜像，然后创建容器，如果一切顺利，运行后，打开浏览器，输入`localhost:8080`就可以看到一个页面，表示容器运行成功。



#### 后续无法上网问题

后续无法上网，暂时解决了，操作如下：

- 启动WMWARE之前确保services.msc里面的WMWARE相关DHCP服务和NAT服务要运行

- 确保虚拟网络编辑器里面有wmnet0，这个是桥接模式，wmnet1（仅主机模式），wmnet8（NAT）模式，都要配置好IPV4地址，一般恢复一下默认设置就可以，确保VMWARE的对应服务是开启的，就会自动通过DHCP分配对应IP
- 虚拟机管理台配置了网卡，选择NAT模式，这里不选自定义模式
- 主机在网络适配器里面找到wmnet8，配置IPV4，IP地址是192.168.X.2，网关是192.168.X.1，子网掩码也是255.255.255.0，然后DNS也配置一下谷歌的8888和8844
- WMWARE里面的网络管理器也配置一下，NAT模式不要启用DHCP（**这里存疑，先改为启用DHCP模式**），然后NAT设置里面网关不要配置错了，要和主机wmnet8适配器里面的IPV4网关地址一样，192.168.X.1，**我们都把网关设置为1，把主机设置为2，VMWARE设置为128**
- 执行`ip addr`查一下ens33的配置，state如果是DOWN，执行`sudo ip link set ens33 up`，启动它
- 写了一个脚本用于处理开始后的IP地址问题，很奇怪每次启动后ip addr的ens33都不显示IPV4地址，所以写了一个脚本：

```sh
#!/bin/bash
echo "begin config"
sudo ip addr add 192.168.139.128/24 dev ens33
sudo ip link set ens33 up
sudo ip route add default via 192.168.139.1
read -p "done, press any key to exit"
```

以后每次开机后运行一下，需要保证DHCP服务和NAT服务都要运行。

- 然后通过`ip route`检查一下，上述脚本把IPV4，ens33启动，添加路由都做了
- 然后检查一下，执行`sudo apt update`，如果有问题，比如解析不到域名，那么说明是DNS的问题了，需要改DNS
- 核心是修改`/etc/resolv.conf`，但是这个文件有可能会被sytemd-resolved服务给删除，因此需要先停止systemd-resolved服务：

```
systemctl status systemd-resolved // 查看服务是否启动，如果是active runnning，则执行下面的命令
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved
```

- 建议先删除旧文件，执行`sudo rm -f /etc/resolv.conf`

- 然后重新创建文件

- ```
  sudo touch /etc/resolv.conf
  sudo chmod 644 /etc/resolv.conf
  ```

- 然后编辑文件：`sudo nano /etc/resolv.conf`，添加`nameserver 8.8.8.8  nameserver 8.8.4.4`，注意换行

- 最后保存，然后重启一下，因为systemd-resolved已经停止了，所以不会去删除这个问题，那么就会以这个文件为准

到此就差不多了，以后每次开机还是要运行一下脚本，不然还是会有问题，一些奇怪的小问题就不去管它了，开机运行脚本后面改成了通过快捷方式去做。
