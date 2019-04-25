# IIS7.0/7.5解析漏洞

> IIS7.0/7.5本身没有解析漏洞，但是当IIS通过[FastCGI](https://baike.baidu.com/item/FastCGI)解析PHP语言时，由于FastCGI配置中的cgi.fix_pathinfo默认为1（即开启状态），导致在对文件进行解析时形成解析漏洞，因此，IIS7.0/7.5解析漏洞实际上是PHP FastCGI解析漏洞

## 漏洞原理

> 如果IIS7.0/7.5通过FastCGI的方式支持PHP解析，当访问`http://172.16.16.24/index.jpg/xx.php`时，$fastcgi_script_name会被设置为 `index.jpg/xx.php`,如果开启cgi.fix_pathinfo，PHP会认为SCRIPT_FILENAME是`index.jpg`，而`xx.php`是PATH_INFO，所以就会将`index.jpg`作为PHP文件来进行解析，从而形成解析漏洞

:pencil:**Thinking**  
> 漏洞利用：通过上传点，上传一个包含木马的任意格式的文件，在菜刀连接时或者解析时，在完整访问路径下添加`/*.php`，该文件就会以PHP的形式进行解析

## 测试环境

- **操作系统**：Windows Server 2008 R2
- **应用服务**：IIS 7.5
- **解析语言**：PHP 7.2.17
- **测试时间**：2019/4/25

## 测试过程

### 基本测试
















> 通过上传点，上传一个图片木马，然后访问该图片木马，让其在本目录下生成一个shell.php，通过该方法可以绕过上传白名单检测，文件内容检测

### 制作图片木马










### 测试过程

如图所示，在网站的根目录下建立了一个xx.cer的文件夹，在xx.asa的文件夹下又建立了一个1.png的文件

![iis6_0001](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0001.PNG)

当在页面执行1.png文件时，IIS会把1.png当做asp文件进行解析

![iis6_0002](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0002.PNG)

当在以上四种任意一种目录下，写入一句话木马，发现可以正常连接

![iis6_0003](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0003.PNG)

![iis6_0004](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0004.PNG)

:pencil:**Thinking**

> 本文对以上四种类型命名的文件夹均进行了测试，发现文件夹中的任意类型的文件均能以asp类型的文件进行解析。如果在文件上传时，能够重写文件夹上传路径或者新建文件夹，那么就能随便写入一个asp内容的木马文件（文件的后缀无所谓），获取网站webshell

## 2.后缀解析漏洞

### 理论

> 后缀解析漏洞存在两种类型：

- **类型1**：当文件的后缀为以下几种类型时，IIS会把该文件作为asp类型的文件进行解析

```
*.asp
*.asa
*.cer
*.cdx   （该类型需要添加认证才能解析）
```

:pencil:**Thinking**

> 1.asa文件在被IIS解析时，会被正常当做1.asp进行解析。所以当上传端设置黑名单不让上传asp类型文件时，可以上传asa/cer类型的木马，仍然可以正常解析

- **类型2**：当文件的后缀为以下几种类型时，分号后面的内容不解析

```
*.asp;*
*.asa;*
*.cer;*
*.cdx;*   （该类型需要添加认证才能解析）
```

:pencil:**Thinking**

> 1.asp;2.jpg文件在被解析时，IIS不解析;后面的内容，会当做1.asp文件进行解析。所以当上传端无论设置黑名单还是白名单，都可以通过构造后缀进行绕过。当设置限制asp等类型的黑名单时，可以上传类似1.asp;.jpg文件；当设置只能上传png等类型的白名单时，可以上传类似1.asp;.png文件

### 测试过程

![iis6_0005](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0005.PNG)

![iis6_0006](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0006.PNG)

![iis6_0007](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0007.PNG)

![iis6_0008](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_5.x_6.0/image/iis6_0008.PNG)





















