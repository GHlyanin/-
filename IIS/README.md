# IIS文件解析漏洞

> 目前IIS解析漏洞主要概括为以下两种情况

- **IIS5.x/6.0解析漏洞**
- **IIS7.0/7.5解析漏洞**

## IIS5.x/6.0解析漏洞

> IIS5.x和IIS6.0存在两种解析漏洞，目录解析漏洞和后缀解析漏洞，本次实验主要在Windows Server 2003 R2 + IIS6.0的环境下对这两种漏洞进行测试，IIS5.x版本不做测试。

- **操作系统**：Windows Server 2003 R2
- **应用服务**：IIS6.0/ASP

### 1.目录解析漏洞

------

> 原理：当在网站目录下建立如下任意一种文件夹，文件夹中任何扩展名的文件都会被IIS当做asp类型的文件进行解析。

```
*.asp
*.asa
*.cer
*.cdx
```

如图所示，在网站的根目录下建立了一个xx.cer的文件夹，在xx.asa的文件夹下又建立了一个1.png的文件

![iis6_0001](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/iis6_0001.PNG)

当在页面执行1.png文件时，IIS会把1.png当做asp文件进行解析

![iis6_0002](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/iis6_0002.PNG)

当在以上四种任意一种目录下，写入一句话木马，发现可以正常连接

![iis6_0003](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/iis6_0003.PNG)

![iis6_0004](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/iis6_0004.PNG)

 **注：**本次实验对以上四种类型的文件夹均进行了测试，发现文件夹中的任意类型的文件均能以asp类型的文件进行解析。如果在文件上传时，能够重写文件夹上传路径或者新建文件夹，那么就能随便写入一个asp内容的木马文件（文件的后缀无所谓），获取网站webshell。

### 2.后缀解析漏洞

------

> 后缀解析漏洞存在两种情况

- **分号后面的内容不解析**























