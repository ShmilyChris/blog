# WinRM利用

**关于WinRM**

* WinRM执行WS-Management协议(用于远程软件和硬件管理的web管理协议)实现远程管理
* 端口为5985
* 只有在Windows server 2008以上的版本服务器WinRM服务才会自动启动
* 启用WinRM服务后，防火墙会自动放行

**WinRM执行远程命令**

* Winrs：运行远程执行命令的命令行工具，利用WS-Management协议
* Winrm: 内置系统管理命令行工具，允许管理配置本机的WinRM服务

Winrs error:WinRM客户端无法处理该请求解决

* 传输HTTPS或者目标位于TrustedHost列表中，并显示凭证
* 使用Winrm.cmd配置TrustedHosts

手动将目标的IP地址添加到客户端信任列表(TrustedHosts)中

```
Winrm set winrm/config/client @{TrustedHosts="192.168.0.100"}
```

信任所有主机

```
Set-Item WSMan:localhost\client\trustedhosts -value *
```

**Winrs**

通过Winrs在远程主机上执行命令

```
winrs -r:http://10.10.10.22:5985 -u:Administrator -p:P@ssW0rd! "whoami"
```

通过Winrs获取远程主机的交互式命令

```
winrs -r:http://10.10.10.22:5985 -u:Administrator -p:P@ssW0rd! "cmd"
```

**Winrm.cmd**

* Winrm.cmd允许WMI对象通过WinRM传输进行远程交互

启动notepad.exe进程

```
winrm invoke create wmicimv2/win32_process -SkipCAcheck -skipCNcheck @{commandline="notepad.exe"} -r:http://192.168.0.104:5985 -u:Administrator -p:P@ssW0rd!
```

**Powershell下的WinRM利用**

设置连接要求

```
$User = "Administrator"
$Password = ConvertTO-SecureString -String "P@ssW0rd!" -AsplainText -Force
$Cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User,$Password
```

利用WinRm启动一个交换式会话

```
New-PSSession -Name WinRM1 -Computername 192.168.0.104 -Credential $Cred -port 5985
```

**Evil-Winrm**

```
https://github.com/Hackplayers/evil-winrm
```
