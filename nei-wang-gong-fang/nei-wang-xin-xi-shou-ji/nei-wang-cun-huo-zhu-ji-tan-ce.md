# 内网存活主机探测

#### 基于ICMP

bat一句话探测：

```、
for /L %i in (1,1,254) DO @ping -w 1 -n 1 10.10.10.%i | findstr "TTL="
```

#### 基于NetBIOS

工具：nbtscan(https://sectools.org/tool/nbtscan/)

```
nbtscan -r 192.168.144.1/24
```

#### 基于UDP

Kali自带工具：Unicornscan

```
Unicornscan -mU <ip>
```

#### 基于ARP

方法1：工具arp-scan(kali 自带)

```
arp-scan -I eth0 --srcaddr=DE:AD:BE:EF:CA:FE 192.168.86.0/24
```

方法2：PS脚本探测

```
$IPs = (1..254 | ForEach-Object { "192.168.31.$_"})
$results = @()

foreach ($ip in $IPs) {
    $ping = New-Object System.Net.NetworkInformation.Ping
    $reply = $ping.Send($ip)
    if ($reply.Status -eq "Success") {
        $results += $ip
    }
}

$results

```

#### 基于SMB

后渗透工具：CrackMapExec(https://github.com/Porchetta-Industries/CrackMapExec)

```
CrackMapExec smb 192.168.1.0/24
```

#### 最优解

Angry IP Scanner(https://angryip.org/download/)

<figure><img src="../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>
