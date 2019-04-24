# IIS文件解析漏洞

> 目前IIS解析漏洞主要概括为以下两种情况

- **IIS5.x/6.0解析漏洞**
- **IIS7.0/7.5解析漏洞**

## IIS5.x/6.0解析漏洞

> IIS5.x和IIS6.0存在两种解析漏洞，目录解析漏洞和后缀解析漏洞，本次实验主要在Windows Server 2003 R2 + IIS6.0的环境下对这两种漏洞进行测试，IIS5.x版本不做测试。

- **操作系统**：Windows Server 2003 R2
- **应用服务**：IIS6.0/ASP

### 1.目录解析漏洞

**理论**

> 当在网站目录下，建立如下命名的任意一种文件夹，文件夹中任何扩展名的文件都会被IIS当做asp类型的文件进行解析。

```
*.asp
*.asa
*.cer
*.cdx
```

**实验**


如图所示，在网站的根目录下建立了一个xx.cer的文件夹，在xx.asa的文件夹下又建立了一个1.png的文件

![iis6_0001](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/iis6_0001.PNG)

当在页面执行1.png文件时，IIS会把1.png当做asp文件进行解析

![iis6_0002](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/iis6_0002.PNG)

当在以上四种任意一种目录下，写入一句话木马，发现可以正常连接

![iis6_0003](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/iis6_0003.PNG)

![iis6_0004](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/iis6_0004.PNG)

**注**：本次实验对以上四种类型命名的文件夹均进行了测试，发现文件夹中的任意类型的文件均能以asp类型的文件进行解析。如果在文件上传时，能够重写文件夹上传路径或者新建文件夹，那么就能随便写入一个asp内容的木马文件（文件的后缀无所谓），获取网站webshell。

### 2.后缀解析漏洞

**理论**

> 后缀解析漏洞存在两种类型：

- **类型1**：当文件的后缀为以下几种类型时，IIS会把该文件作为asp类型的文件进行解析

```
*.asp
*.asa
*.cer
*.cdx   （该类型需要添加认证才能解析）
```

**注**：1.asa文件在被IIS解析时，会被正常当做1.asp进行解析。所以当上传端设置黑名单不让上传asp类型文件时，可以上传asa/cer类型的木马，仍然可以正常解析

- **类型2**：当文件的后缀为以下几种类型时，分号后面的内容不解析

```
*.asp;*
*.asa;*
*.cer;*
*.cdx;*   （该类型需要添加认证才能解析）
```

**注**：1.asp;2.jpg文件在被解析时，IIS不解析;后面的内容，会当做1.asp文件进行解析。所以当上传端无论设置黑名单还是白名单，都可以通过构造后缀进行绕过。当设置限制asp等类型的黑名单时，可以上传类似1.asp;.jpg文件；当设置只能上传png等类型的白名单时，可以上传类似1.asp;.png文件。

**实验截图**























