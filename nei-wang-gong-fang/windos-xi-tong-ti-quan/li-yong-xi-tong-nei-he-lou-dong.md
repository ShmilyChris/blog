# 利用系统内核漏洞

## WES-NG提权

获取系统信息

```
systeminfo > systeminfo.txt
```

wes-ng查找存在的漏洞并查找exp

```
python3 wes.py systeminfo.txt -i 'Elevation of Privilege' --exploits-only
```

<figure><img src="../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>
