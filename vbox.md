# 命令行下使用 VirtualBox 安装 NOI Linux

## 0 缘起

这两天模拟赛越发频繁了...

Windows 有时候很睿智，Lemon 时间波动特别大，Cena ... 不说了大家都懂

总而言之，与其和微软斗智斗勇，不如早日更换评测环境

然而如果使用现今的发行版，有一个很有意思的问题...

CCF 的评测环境有些久远，而且还是 32 位的

所以果然还是 NOI Linux 吧～

然而机房并没有空闲的电脑能够完成这个任务，虚拟机大家又跑不起

我的眼光落到了正在运行着 NovaOJ 的工作站上

就决定是你了！

## 1 VirtualBox 安装

> 系统环境：   
> `Linux hostname 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u1 (2019-09-20) x86_64 GNU/Linux`  
> 以下命令默认使用 `root` 权限

### 1.1 安装标准包

直接从官网...

并没有 Debian 10 的包

不过可以添加软件源

```bash
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
echo 'deb [arch=amd64] https://download.virtualbox.org/virtualbox/debian buster contrib' >> /etc/apt/sources.list
apt-get update
apt-get install virtualbox-6.0
```

### 1.2 安装扩展包

你可以从[ 官方站点 ](https://www.virtualbox.org/wiki/Downloads)的`VirtualBox *** Oracle VM VirtualBox Extension Pack` 一栏获取最新版的下载链接

```bash
wget https://download.virtualbox.org/virtualbox/6.0.14/Oracle_VM_VirtualBox_Extension_Pack-6.0.14.vbox-extpack
VBoxManage extpack install ./Oracle_VM_VirtualBox_Extension_Pack-6.0.14.vbox-extpack
```

## 2 获取 NOI Linux 安装包

自己去[ NOI 官网 ](http://www.noi.cn)找去...

## 3 创建虚拟机

```bash
vm_name="noi_linux"
vm_hard="/oi/noi_linux/hard.vdi"
noi_linux_iso="/home/test/noilinux-1.4.1.iso"
vboxmanage createvm --name $vm_name --ostype noi_linux --register
VBoxManage createvdi --filename $vm_hard --size 20480
vboxmanage modifyvm $vm_name --memory 4096 --vram 256 --hwvirtex on
vboxmanage storagectl "$vm_name" --name "SATA Controller" --add sata --hostiocache on --bootable on
vboxmanage storagectl "$vm_name" --name "IDE Controller" --add ide --controller PIIX4 --hostiocache on --bootable on
vboxmanage storageattach $vm_name --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium $vm_hard
vboxmanage storageattach $vm_name --storagectl "IDE Controller" --port 1 --device 1 --type dvddrive --medium $noi_linux_iso
vboxmanage modifyvm $vm_name --vrde on --vrdeport 5688
VBoxManage modifyvm $vm_name --cpus 2
vboxmanage modifyvm $vm_name --nic1 nat
VBoxHeadless -s $vm_name
```

这样子你的虚拟机就在 5688 端口上建立一个远程桌面

使用 Windows 的 远程桌面链接或者 `rdsktop` 即可

接下来大家都会安装，问题在与如何链接和上传文件

## 4 创建端口映射并建立 WebDAV

短暂的思考后，我们可以建立 WebDAV 并配以适当的权限控制，来完成这一目标

### 4.1 端口映射

```bash
vboxmanage controlvm $vm_name natpf1 “web,ssh,,2012,,22”
vboxmanage controlvm $vm_name natpf1 “web,tcp,,2022,,80”
```

### 4.2 配置 WebDAV

在 NOI Linux 下执行

```bash
apt-get install apache2 apache2-utils
a2enmod dav
a2enmod dav_fs
```

编辑 `/etc/apache2/sites-enabled/000-default.conf` 文件

```apache
<VirtualHost *:80>
	DocumentRoot /home/noilinux/oi
	Alias / /home/noilinux/oi/
	<Directory "/home/noilinux/oi">
		Dav On
		Options +Indexes
		IndexOptions FancyIndexing
		AddDefaultCharset UTF-8
		AuthType Basic
		AuthName "NOI Linux Test Env"
		AuthUserFile /var/www/passwd.dav
		Order allow,deny
		Allow from all
		<LimitExcept GET PUT POST DELETE MOVE COPY OPTIONS>
			Require user admin
		</LimitExcept>
		<LimitExcept PUT POST DELETE MOVE OPTIONS>
			Require user guest
		</LimitExcept>
	</Directory>

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

其中 `LimitExcept` 项是权限控制

```bash
htpasswd -c /var/www/passwd.dav guest
htpasswd -c /var/www/passwd.dav admin
service apache2 restart
```

使用 WebDAV 访问工具访问 `ip:2022` 即可上传文件至 `/home/noi_linux/oi` 目录下

请确保 `/home/noi_linux/oi` 目录存在且可读写

## 5 虚拟机自启动

修改 `/etc/systemd/system/vbox@.service` 文件

```bash
[Unit]
Description=VirtualBox Machine Service - %I
After=network.target
Wants=network.target

[Service]
Type=simple
User=woshiluo
ExecStart=VBoxHeadless -s %I
ExecStop=vboxmanage controlvm %I poweroff
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

执行

```bash
systemctl start vbox@$vm_name.service 
systemctl enable vbox@$vm_name.service 
```


## 6 结

果然命令行是可以解决所有问题的www

之后编译个 Lemon Plus 即可

不过还是希望 CCF 升级一下评测环境啊

毕竟 32 位系统也开始逐渐被抛弃了

本文提供了一种学校内模拟 NOI Linux 评测环境的方法，并提供了一种较为合理的文件传输及权限控制方法

对于有服务器的学校是一种较为优秀的解决方案

## 7 引用

本文在撰写时参考了下列网页或博客，特此写出，部分内容来源不清，故未写出

- [VBoxManage 主机与宿主机之间实现端口映射](https://blog.csdn.net/a85820069/article/details/79038119)
- [CentOS7命令行安装VirtualBox并安装虚拟机](https://zhuanlan.zhihu.com/p/29759641)
