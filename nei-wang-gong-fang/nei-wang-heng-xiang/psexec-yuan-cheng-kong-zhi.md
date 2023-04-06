# PsExec远程控制

* 微软官方的Windows远程控制工具
* 通过SMB连接到服务端的Admin$并释放psexecsvc.exe的二进制文件，然后注册PSEXECSVC的服务
* PsExec远程操作的前置条件：远程主机开启了Admin$共享、远程主机放行445或未开启防火墙

```
[[accepteula禁止弹出许可证对话框；-u指定用户名，-p指定密码，-s以system权限启动进程]]
PsExec.exe -accepteula \\10.10.10.22 -u Administrator -p P@ssW0rd! -s cmd.exe

建立IPC$连接的基础上可直接连接
Psexec.exe -accepteula \\10.10.10.22 cmd.exe
```
