# IIS5.x/6.0解析漏洞

> IIS5.x和IIS6.0存在两种解析漏洞，目录解析漏洞和后缀解析漏洞，本次测试主要在Windows Server 2003 R2 + IIS6.0的环境下对这两种漏洞进行测试，IIS5.x版本不做测试

## 测试环境

- **操作系统**：Windows Server 2003 R2
- **应用服务**：IIS 6.0
- **解析语言**：ASP
- **测试时间**：2019/4/24

## 测试内容

- [目录解析漏洞](#0x01-目录解析漏洞)
- [后缀解析漏洞](#0x02-后缀解析漏洞)

## 测试过程

### 0x01 目录解析漏洞

**理论**

> 当在网站目录下，建立如下命名的任意一种文件夹，文件夹中任何扩展名的文件都会被IIS当做ASP类型的文件进行解析

```
*.asp
*.asa
*.cer
*.cdx
```

**测试**

> 如图所示，在网站的根目录下建立了一个`xx.cer`的文件夹，在`xx.cer`的文件夹下又建立了一个`1.png`的文件

![iis6_0001](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0001.PNG)

> 当在页面执行`1.png`文件时，IIS会把`1.png`当做PHP文件进行解析

![iis6_0002](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0002.PNG)

> 当在以上四种任意一种目录下，写入一句话木马，发现可以正常连接

![iis6_0003](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0003.PNG)

![iis6_0004](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0004.PNG)

:pencil:**Thinking**

> 本文对以上四种类型命名的文件夹均进行了测试，发现文件夹中的任意类型的文件均能以ASP类型的文件进行解析。如果在文件上传时，能够重写文件夹上传路径或者新建文件夹，那么就能随便写入一个ASP内容的木马文件（文件的后缀无所谓），获取网站webshell

### 0x02 后缀解析漏洞

**理论**

> 后缀解析漏洞存在两种类型：

- **类型1**：当文件的后缀为以下几种类型时，IIS会把该文件作为ASP类型的文件进行解析

```
*.asp
*.asa
*.cer
*.cdx   （该类型需要添加认证才能解析）
```

:pencil:**Thinking**

> `1.asa`文件在被IIS解析时，会被正常当做`1.asp`进行解析。所以当上传端设置黑名单不让上传ASP类型文件时，可以上传`asa/cer`类型的木马，仍然可以正常解析

- **类型2**：当文件的后缀为以下几种类型时，分号后面的内容不解析

```
*.asp;*
*.asa;*
*.cer;*
*.cdx;*   （该类型需要添加认证才能解析）
```

:pencil:**Thinking**

> `1.asp;2.jpg`文件在被解析时，IIS不解析;后面的内容，会当做`1.asp`文件进行解析。所以当上传端无论设置黑名单还是白名单，都可以通过构造后缀进行绕过。当设置限制ASP等类型的黑名单时，可以上传类似`1.asp;.jpg`文件；当设置只能上传PNG等类型的白名单时，可以上传类似`1.asp;.png`文件

**测试**

![iis6_0005](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0005.PNG)

![iis6_0006](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0006.PNG)

![iis6_0007](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0007.PNG)

![iis6_0008](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0008.PNG)





















