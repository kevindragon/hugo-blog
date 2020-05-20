---
title: 从virtualbox创建vagrant box
date: 2016-08-24 22:40:54
tags: [vagrant, virtualbox]
---

有的时候vagrant官网提供的boxes可能并不是想要的，有些包含了太多东西，有些box又没有需要的。如果你的开发团队需要这样的一个box，那么你就应该自己动手创建一个。其实创建的过程挺简单的，只要每个步骤操作正确。

Follow下面步骤：

1、安装vagrant

2、安装virtualbox

3、在virtualbox创建一个linux (centos)，假设名字叫centos

4、创建好centos后，还需要安装virtualbox的guest增强插件。安装之前需要在centos安装一些必须的工具：

    # yum install kernel-devel gcc perl
    # ln -s /usr/src/kernels/2.6.18-164.15.1.el5-i686 /usr/src/linux 

上面第二行请把版本号换成你自己安装的centos的内核版本号，你可以通过`uname -a`查看，或者直接到`/user/src/kernels`目录查看即可

5、还需要创建一个`vagrant`账号，密码也是vagrant。注意请把`root`密码也设置成`vagrant`

    # groupadd vagrant
    # useradd -g vagrant vagrant
    # passwd vagrant

另外还需要设置sudo，因为vagrant默认会使用vagrant账号挂载工作目录到`/vagrant`。还需要设置`tty`，用root账号输入`visudo`，修改centos里面的`/etc/sudoers`注释下面两行

    #Defaults    requiretty
    #Defaults   !visiblepw

添加下面一行到`root    ALL=(ALL)       ALL`下面

    vagrant ALL=(ALL)       ALL

6、检查`guest******.iso`是否加载，如果没有加载的话，请添加一个光驱，并且加载这个iso文件，一般这个iso文件在virtualbox的安装目录可以找到

7、连接上终端，输入下面的命令：

    mount /dev/cdrom /media/cdrom
    cd /media/cdrom
    ./VBoxLinuxAdditions.run

如果没有error说明安装成功，如果有关于X的错误，有可能是你没有安装X环境，如果只是使用命令行，这个也不用管

8、打包

    vagrant package --base centos --output centos.box

`vagrant package`就是vagrant用来打包的命令，`--base`指定你创建的虚拟机的名字，后面`--output`指定导出的box的位置和名字

9、打好包后，自己可以建立一个新的环境测试一下：

    vagrant box add base centos.box
    cd your vagrant workspace dir
    vagrant init base

10、这里要注意的一点是，由于没有找到使用publickey登录的方式，所以必须要在`Vagrantfile`的最后添加用户名和密码登录

    Vagrant.configure("2") do |config|
      # 添加下面两行使用用户名密码登录
      config.ssh.username = "root"
      config.ssh.password = "vagrant"
    end

11、启动

    vagrant up


