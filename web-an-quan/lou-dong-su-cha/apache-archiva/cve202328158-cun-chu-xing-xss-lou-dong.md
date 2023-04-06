# CVE-2023-28158 存储型XSS漏洞

发现时间：2023/3/30

漏洞版本：org.apache.archiva:archiva-web-common@(-∞, 2.2.10)

Apache Archiva是一个用于软件构建和部署的开源项目。该项目受影响版本存在存储型XSS漏洞，由于Archiva未对用户输入的目录名进行过滤或转义，具备登录权限的攻击者可在目录名中注入JavaScript 代码。当用户浏览到包含JavaScript 代码的目录时，目录名中的JavaScript 代码被浏览器执行







