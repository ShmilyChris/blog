# 内网端口扫描

#### 端口扫描

fscan（https://github.com/shadow1ng/fscan）

```
fscan.exe -p 1-65535 -h 192.168.1.1/24  (默认使用全部模块)
```

#### 获取端口Banner信息

Netcat

```
nc -nv <ip>  <port>
```

Nmap

```
nmap --scrtpt=banner -p xx <ip>
```
