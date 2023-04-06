# Potato家族提权

#### 前置知识备忘

1. Potato本质上是通过操纵访问令牌，将已获取的Windows服务账户权限提升到系统SYSTEM权限
2. Potato家族通过滥用Windows服务账户拥有`SeAssignPrimaryTokenPrivilege`和`SeImpersonatePrivilege`特权，将已获取的`NT AUTHORITY\SYSTEM` 账户的访问令牌传入`CreateProcessWIthTokenW`或`CreateProcessAsUserA`函数进行调用，从而在`NT AUTHORITY\SYSTEM`账户的上下文中创建新进程，提升至SYSTEM权限

#### Rotten potato

大致原理：远程过程调用(RPC)通过CoGetInstanceFromIStorage进行NTLM中继,在本地协商得到NT AUTHORITY\SYSTEM账户的安全令牌，在模拟安全令牌后调用CreateProcessWithToken等API来使用指定的令牌来创建进程

利用：https://github.com/breenmachine/RottenPotatoNG

相比旧版工具，直接运行弹system cmd，摆脱meterpreter依赖

#### juicy potato：

原理：与Rotten potato差不多，区别在于juicy potato可以自定义COM对象加载端口，以及根据系统版本更好可用的COM对象

适用于Windows 10 version 1809和Windows server 2019以前的版本

利用：https://github.com/uknowsec/JuicyPotato/blob/main/JuicyPotato-webshell/Bin/JuicyPotato\_x64.exe

#### PrintSpoofer(pipe potato)

原理：利用打印机组件路径检查缺陷，使得高权限服务能连接到测试人员创建的命名管道，导致获取高权限账户的令牌创建新进程

利用：https://github.com/daikerSec/pipePotato

#### sweet potato

集成了上面各类土豆家族的利用方法，从Windows 7到Windows 10 / Server 2019的本地服务到系统权限提升

https://github.com/CCob/SweetPotato

https://github.com/Tycx2ry/SweetPotato\_CS
