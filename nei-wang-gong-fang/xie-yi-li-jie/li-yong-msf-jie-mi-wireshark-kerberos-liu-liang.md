# 利用MSF解密Wireshark Kerberos流量

在学习Kerberos协议的时候，需要解密wireshark Kerberos流量，记录此过程遇到的一些坑，并且如何解决。

## 前置知识

keytab：(key table, 密钥表) 是一个加密文件, 包含受 Kerberos 保护的服务及其在密钥分发中心 (Key Distribution Center, KDC) 中关联的服务主体名称 (service principal name, SPN) 的长期密钥，其用途之一就是解密 Kerberos

NTDS.dit：文件属于ESE (Extensible Storage Engine, 可扩展存储引擎) 数据库文件，Active Directory 的数据库引擎就是[可扩展存储引擎](https://learn.microsoft.com/en-us/windows/win32/extensible-storage-engine/extensible-storage-engine)(Extensible Storage Engine, ESE), 它基于 Exchange 5.5 和 WINS 使用的 Jet 数据库，通俗来讲就是 Jet 数据库引擎

目前主流的两种解密方式：

1. 域控导NTDS.dit\&system.hive -> esedbexport导出NTDS.dit内容-> 使用ntdsxtract/dskeytab.py脚本导出kt，使用这个办法遇到了一个坑，就是ntdsxtract/dskeytab.py脚本，他只支持python2，脚本老久，导出的最后一步遇到了疑难杂症
2. 使用ktpass \`ktpass /crypto All /princ Administrator@DEMO.LOCAL /pass p4\$$w0rd /out demo.keytab /ptype KRB5\_NT\_PRINCIPAL\` 手动一个个导出，这个方法很慢，而且后面还需要将不同的kt整合为一个

## 利用MSF解密Wireshark Kerberos流量

在最新版的MSF>=6.3中，新增了非常多内网模块，其中有这下面两个模块：

* auxiliary/gather/windows\_secrets\_dump
* auxiliary/admin/kerberos/keytab

其中auxiliary/gather/windows\_secrets\_dump模块可以借助SMB帮助我们一键远程提取计算机帐户中NT HASH，如下图

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

归功于MSF拥有强大的数据库功能，auxiliary/gather/windows\_secrets\_dump模块的运行结束支持入库，那么我们就可以利用auxiliary/gather/windows\_secrets\_dump配合auxiliary/admin/kerberos/keytab模块，一键导出DC上的所有账户的keytab

### 实操

启动MSF数据库

```
systemctl start postgresql

msfdb inti
```

启动MSF，检查数据库连接情况，显示字段Connected to msf则表示连接成功

```
db_status
```

利用auxiliary/gather/windows\_secrets\_dump模块一键导出NTHASH并入库

```
use auxiliary/gather/windows_secrets_dump
set rhost xxxx
set smbdomain xxx
set smbuser xxx
set smbpass xxx
run
```

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

原地切换auxiliary/admin/kerberos/keytab模块导出kt

<pre><code>use auxiliary/admin/kerberos/keytab
<strong>run action=EXPORT keytab_file=./1.keytab
</strong></code></pre>

<figure><img src="../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

将导出的kt证书导入到wireshark中：编辑->首选项->Protocols->KRB5

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

如果解密成功, 就会是显示蓝色, 不成功就是黄色

<figure><img src="../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>
