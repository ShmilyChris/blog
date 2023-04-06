# Linux全盘搜索、快速查找文件、收集数据库配置文件

```
find / -type f -regex '.*\.xml\|.*\.php\|.*\.jsp\|.*\.conf\|.*\.bak\|.*\.inc\|.*\.htpasswd\|.*\.inf\|.*\.ini\|.*\.properties' | xargs egrep "jdbc"
```
