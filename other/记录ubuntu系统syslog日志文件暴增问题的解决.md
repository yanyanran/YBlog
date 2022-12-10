# 记录ubuntu系统syslog日志文件暴增问题的解决

这个问题曾在今年年初（1.5）出现过一次，如今是十二月份，这也得是2022首尾呼应了啊（救

这个问题比上次遇到的要复杂一些。首先还是熟悉的系统先弹出了个弹窗提示我磁盘空间只剩200多M了

接着终端df -h，显示我的文件系统/dev/nvme0n1p10被占用百分之百：

![](https://github.com/yanyanran/pictures/blob/main/syslogbug.png?raw=true)

我：.................

尝试：cdc

`sudo su`

`echo " " > /var/log/syslog`

上次就是这么解决的，结果这次不管用了。腾出了几G然后在几分钟后又被快速占满。

终端包管理器下载了个监控磁盘IO使用情况的工具iotop，想实时查看下到底是哪个崽种进程在疯狂写我的磁盘，结果也没看到异常（可能是磁盘写满了也没地方写了）

尝试查看是谁产生的日志：

 `vim /var/log/syslog` 

![](https://github.com/yanyanran/pictures/blob/main/syslogbug2.png?raw=true)

亏贼...

操作：

`cd /var/log`

`sudo -i`

`rm -rf /var/log/*.gz`

`rm -rf /var/log/*.1`

`echo "" > /var/log/dmesg`

`echo "" > /var/log/kern.log`

`echo "" > /var/log/messages`

`echo "" > /var/log/syslog`

`sudo apt-get autoclean`			清理旧版本的软件缓存

`sudo apt-get clean`					清理所有软件缓存

`sudo apt-get autoremove`	 	删除系统不再使用的孤立软件

再看一眼 居然：

![](https://github.com/yanyanran/pictures/blob/main/syslogbug3.png?raw=true)





出现了`pcie`错误，日志文件中大部分的都是同样的东西，诸如`PCIe Bus Error`等。

解决方法如下：

打开终端，修改`/etc/default/grub`引导文件

```javascript
sudo cp /etc/default/grub /etc/default/grub.bak
sudo -H gedit /etc/default/grub
```

打开之后找到以下这句

```javascript
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

然后将其改为

```javascript
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash pci=nomsi"
```

保存关闭grub文件，更新grub引导，并重启

```javascript
sudo update-grub
sudo reboot
```

日志文件恢复正常，不会再大量地记录这方面的错误。