---
title: Nginx中location 匹配规则
date: 2024-05-22 15:20:26
tags:
---

# location 匹配的变量
Nginx 的 location 规则匹配的变量是 $uri, 所以不用管后面的参数 $query_string (或者 $args)

# location 匹配的种类

匹配格式:
```json
location [空格 | = | ~ | ~* | ^~ | @ ] /uri/ {
  ...
}
```
上面匹配格式分为三部分:

1. 修饰符匹配规则
   `[空格 | = | ~ | ~* | ^~ | @ ]`
2. uri匹配规则
   `/uri/`
3. 大括号内的路由转发
```json
{
  ...
}
```

# 修饰符匹配规则
格式:
`[空格 | ^~ |  = | ~ | ~* |@ ]`

接下来解释一下，这些都表示啥意思:
|字符|解释|
|-|-|
|空格|无修饰符的前缀匹配，匹配前缀是 你配置的（比如说你配的是 /aaa） 的url|
|^~|常规字符串匹配（跟空格类似，但是它优先级比空格高）。^表示“非”，即不查询正则表达式。如果匹配成功，并且所匹配的字符串是最长的， 则不再匹配其他location。|
|=|表示精确匹配，如果找到，立即停止搜索并立即处理此请求。|
|~|表示执行一个正则匹配，区分大小写|
|`~*`|表示执行一个正则匹配，不区分大小写。注意，如果是运行 Nginx server 的系统本身对大小写不敏感，比如 Windows ，那么 ~* 和 ~ 这两个表现是一样的|
|@|用于定义一个 Location块，且该块不能被外部Client 所访问，只能被Nginx内部配置指令所访问，比如try_files 或 error_page|

## 修饰符的匹配顺序

1. 优先查找精确匹配，精确匹配 (=) 的 location 如果匹配请求 URI 的话，此 location 被马上使用，匹配过程结束。
2. 接下来进行常规字符串匹配(空格 和 ~^), 如果发现匹配最长的那个是 ^~ 前缀, 那么也停止搜索并且马上使用，匹配过程结束。 否则继续往下走。
3. 如果常规字符串匹配失败，或者匹配的最长字符串不是 ^~ 前缀 (比如是空格匹配)，那么继续搜索正则表达式匹配， 这时候就根据在配置文件定义的顺序，取最上面的配置(正则匹配跟匹配长度没关系，只跟位置有关系，只取顺序最上面的匹配)
4. 如果第三步找到了，那么就用第三步的匹配，否则就用第二步的匹配(字符匹配最长的空格匹配)

简单的来说就是顺序如下:
`精确匹配 > 字符串匹配( 长 > 短 [ 注: ^~ 匹配则停止匹配 ]) > 正则匹配( 上 > 下 )`

换成符号的优先级就是:
`[=] > [^~] > [~/~*] > [空格]`

**注意几个细节:**
1. 常规字符串匹配类型。是按前缀匹配(从根开始)。 而正则表达式匹配是包含匹配，只要包含就可以匹配
2. ~和~*的优先级一样，取决于在配置文件中的位置，最上面的为主，跟匹配的字符串长度没关系，所以在写的时候，应该越精准的要放在越前面
3. 空格匹配和^~都是字符串匹配，所以如果两个后面的匹配字符串一样，是会报错的，因为 nginx 会认为两者的匹配规则一致，所以会有冲突
4. ^~, =, ~, ~* 这些修饰符和后面的 URI 字符串中间可以不使用空格隔开(大部分都是用空格隔开)。但是 @ 修饰符必须和 URI 字符串直接连接。

## 例子
```python
location / {
  return 200 '404';
}

location ~ /hello {
  return 200 '1';
}

location ^~ /hello {
  return 200 '2';
}

location = /hello {
  return 200 '3';
}

location ~* /hello {
  return 200 '4';
}
```
这个是测试结果

```python
[root@VM_156_200_centos ~]# curl 127.0.0.1/hello  #精确匹配，直接结束
3
[root@VM_156_200_centos ~]# curl 127.0.0.1/hello11 #字符串匹配，并且最大长度的匹配是 ^~,直接结束
2
[root@VM_156_200_centos ~]# curl 127.0.0.1/hello/22 #字符串匹配，并且最大长度的匹配是 ~^,直接结束
2
[root@VM_156_200_centos ~]# curl 127.0.0.1/11/hello/ #字符串不匹配(前缀匹配)，正则匹配有两个，取最上面的那个
1
[root@VM_156_200_centos ~]# curl 127.0.0.1/11/Hello/ #字符串不匹配(前缀匹配)，正则匹配有一个(大小写不敏感)，取最上面的那个
4
[root@VM_156_200_centos ~]# curl 127.0.0.1/11/Hell #都不匹配，有设置通用匹配，取通用匹配
404
```

配置文件配置:
```python
location /images/test.png {
  return 200  '1';     
}        

location ^~ /images/ { 
  return 200  '2';      
}    

location ~ /images/ {        
  return 200  '3';   
} 

location ~ /images/test.png {        
  return 200  '4';   
}
```
这个是测试结果

```python
[root@VM_156_200_centos ~]#  curl http://127.0.0.1/images/test.png 
3
[root@VM_156_200_centos ~]#  curl http://127.0.0.1/images/1 
2
```
第一个返回 3 是因为本例是 普通匹配和正则匹配都存在， 并且因为 ^~ 匹配不是最长的话，那么就取 正则匹配， 正则匹配满足条件的有两个， 取最上面那个， 所以是 3， (本例的字符串最长匹配是空格匹配)。(对于本例来说，如果去掉后面的两个正则匹配，那么返回的就是 1， 因为空格匹配的字符串是最长的)
第二个返回 2 是因为普通匹配和正则匹配都存在，但是这个^~ 匹配是最长的，所以就是 2。

普通字符串的匹配冲突
如果我这样子写:
```python
location /images/test.png {
     return 200  '1';     
 }        

 location ^~ /images/test.png { 
      return 200  '2';      
 }  
```

这时候我 reload 是会报错的:

```python
root@VM_156_200_centos sbin]# ./nginx -s reload
nginx: [emerg] duplicate location "/images/test.png" in /usr/local/nginx/conf/nginx.conf:20
```

## 辟谣 ~ 比 ~* 优先级高

之前也有在网上看到这种说法，就是匹配顺序的时候，如果都匹配了，那么 ~ 比 ~* 优先级高，其实这个说法是错误的，这哥俩并没有谁比谁高贵，根据上面的匹配顺序，如果都是正则匹配，那么就是谁排在前面，就采用谁。 做个实践, 我的执行顺序是这样子的:
```python
location ~* \.jpG$ {
 return 200  '1';
}

location ~ \.jpg$ {
 return 200  '2';
}
```
我的测试结果如下:

```python
[root@VM_156_200_centos ~]# curl 127.0.0.1/1.jpg
1
[root@VM_156_200_centos ~]# curl 127.0.0.1/1.JPG
1
```
只要有匹配，肯定是最上面的那个 1。

## 辟谣之  修饰符 包含 !~ 和 !~*
我看到有些网上的文章说 nginx 的 location modifier 还包含 !~ 和 !~* 这两个，其实是不对的(也有可能是旧版本的，至少我的最新版本的 nginx 不支持)， 如果你这样子:
```python
location !~ \.(gif|jpg)$ {
  return 200  '1';
}
```

那么在 reload 的时候， nginx 会报这个错误:
```python
nginx: [emerg] invalid location modifier "!~" in /usr/local/nginx/conf/nginx.conf:25
```
不过因为 Nginx 的正则是使用PCRE（Perl Compatible Regular Expressions）, 所以我们可以这样子写来达到我们想要的目的:

```python
location ~ \.*(?<!(gif|jpg))$ {
 return 200  '1';
}
```
做个实践，假设我的路由是这样子: 如果后缀含有 gif 或者是 jpg， 那么就会返回 200， 否则就会返回 404
```python
location / {
  return 200 '404';
}

location ~ \.(gif|jpg)$ {
 return 200  '200';
}
```
测试结果如下:

```python
[root@VM_156_200_centos ~]# curl 127.0.0.1/1.gif
200
[root@VM_156_200_centos ~]# curl 127.0.0.1/1.jpg
200
[root@VM_156_200_centos ~]# curl 127.0.0.1/1.js
404
[root@VM_156_200_centos ~]# curl 127.0.0.1/1.css
404
```
这个结果是对的。 那么就换成，如果后缀不是 gif 或者是 jpg，那么才返回 200， 否则就返回 404 (相当于上述的 !~):
```python
location / {
  return 200 '404';
}
location ~ \.*(?<!(gif|jpg))$ {
  return 200  '200';
}
```
可以看到，同样的结果，结果相反了:
```python
[root@VM_156_200_centos ~]# curl 127.0.0.1/1.gif
404
[root@VM_156_200_centos ~]# curl 127.0.0.1/1.jpg
404
[root@VM_156_200_centos ~]# curl 127.0.0.1/1.js
200
[root@VM_156_200_centos ~]# curl 127.0.0.1/1.css
200
```
所以是可以实现这种效果的。

## @前缀的命名匹配
这个主要是内部定义一个 location 块，举个例子，因为 location 只验证 uri，参数是没有验证的，如果我们要验证参数，并且要根据不同的参数来进行不同的操作的话，就可以用这个内部定义块，举个例子:
```python
location / {
    error_page 418 = @queryone;
    error_page 419 = @querytwo;
    error_page 420 = @querythree;

    if ( $args ~ "service=one" ) { return 418; }
    if ( $args ~ "service=two" ) { return 419; }
    if ( $args ~ "service=three" ) { return 420; }

    # do the remaining stuff
    # ex: try_files $uri =404;

  }

  location @queryone {
    return 200 'do stuff for one';
  }

  location @querytwo {
    return 200 'do stuff for two';
  }

  location @querythree {
    return 200 'do stuff for three';
  }
```
测试结果如下:

```python
[root@VM_156_200_centos ~]#  curl http://127.0.0.1/?service=one
do stuff for one

[root@VM_156_200_centos ~]#  curl http://127.0.0.1/?service=two
do stuff for two

[root@VM_156_200_centos ~]#  curl http://127.0.0.1/?service=three
do stuff for three
```

# location uri匹配规则
根据前面的符号，这里可以填写精确到 path 路径，也可以填正则表达式，Nginx 用的是 PCRE正则表达式语法，下表是在PCRE中元字符及其在正则表达式上下文中的行为的一个完整列表：
|字符|描述|
|-|-|
|\|将下一个字符标记为一个特殊字符、或一个原义字符、或一个向后引用、或一个八进制转义符。例如，\\n 匹配 \n。\n 匹配换行符。序列 \\ 匹配 \ , 而 \( 则匹配 (。即相当于多种编程语言中都有的转义字符的概念。|
|^|匹配输入字符串的开始位置。如果设置了 RegExp 对象的 Multiline 属性，^ 也匹配\n 或 \r 之后的位置。|
|$|匹配输入字符串的结束位置。如果设置了 RegExp 对象的 Multiline 属性，$ 也匹配\n 或 \r 之前的位置。|
|`*`|匹配前面的子表达式零次或多次。例如，zo* 能匹配 z 以及zoo。 * 等价于 {0,}。|
|+|匹配前面的子表达式一次或多次。例如，zo+ 能匹配 zo 以及 zoo，但不能匹配 z。+ 等价于 {1,}。|
|?|匹配前面的子表达式零次或一次。例如，do(es)? 可以匹配 does 或 do。? 等价于 {0,1}。|
|{n}|n 是一个非负整数。匹配确定的n次。例如，o{2} 不能匹配 Bob 中的 o，但是能匹配 food 中的两个o。|
|{n,}|n 是一个非负整数。至少匹配n次。例如，o{2,} 不能匹配 Bob 中的 o，但能匹配 foooood 中的所有o。o{1,} 等价于 o+ 。o{0,} 则等价于 o*。|
|{n,m}|m和n均为非负整数，其中n<=m。最少匹配n次且最多匹配m次。例如，o{1,3} 将匹配 fooooood 中的前三个o。o{0,1} 等价于o?。 请注意在逗号和两个数之间不能有空格。|
|?|当该字符紧跟在任何一个其他限制符（*,+,?，{n}，{n,}，{n,m}）后面时，匹配模式是非贪婪的。非贪婪模式尽可能少的匹配所搜索的字符串，而默认的贪婪模式则尽可能多的匹配所搜索的字符串。例如，对于字符串oooo，o+? 将匹配单个 o，而 o+ 将匹配所有 o。|
|.|匹配除\n和\r之外的任何单个字符。要匹配包括\n和\r在内的任何字符，请使用像[\s\S]的模式。|
|(pattern)|匹配pattern并获取这一匹配。所获取的匹配可以从产生的 Matches 集合得到，在 VBScript 中使用 SMatches 集合，在JScript中则使用 0…0…0…9 属性。要匹配圆括号字符，请使用\( 或 \)。|
|(?:pattern)|匹配 pattern 但不获取匹配结果，也就是说这是一个非获取匹配，不进行存储供以后使用。这在使用或字符 "(|)" 来组合一个模式的各个部分是很有用。例如"industr(?:y|ies)" 就是一个比 "industry|industries" 更简略的表达式。|
|(?=pattern)|非获取匹配，正向肯定预查，在任何匹配 pattern 的字符串开始处匹配查找字符串。这是一个非获取匹配，也就是说，该匹配不需要获取供以后使用。例如，"Windows(?=95|98|NT|2000)" 能匹配 Windows2000 中的 Windows，但不能匹配 Windows3.1 中的Windows。 预查不消耗字符，也就是说，在一个匹配发生后，在最后一次匹配之后立即开始下一次匹配的搜索，而不是从包含预查的字符之后开始。|
|(?!pattern)|非获取匹配，正向否定预查，在任何不匹配 pattern 的字符串开始处匹配查找字符串。这是一个非获取匹配，也就是说，该匹配不需要获取供以后使用。例如"Windows(?!95|98|NT|2000)" 能匹配 Windows3.1 中的 Windows，但不能匹配Windows2000 中的Windows。 预查不消耗字符，也就是说，在一个匹配发生后，在最后一次匹配之后立即开始下一次匹配的搜索，而不是从包含预查的字符之后开始|
|(?<=pattern)|非获取匹配，反向肯定预查，与正向肯定预查类似，只是方向相反。例如，"(?<=95|98|NT|2000)Windows" 能匹配 2000Windows 中的 Windows，但不能匹配 3.1Windows 中的 Windows。|
|(?<!pattern)|非获取匹配，反向否定预查，与正向否定预查类似，只是方向相反。例如 "(?<!95|98|NT|2000)Windows" 能匹配 3.1Windows 中的 Windows，但不能匹配 2000Windows 中的 Windows。|
|x|y|匹配 x 或 y。例如，"z|food" 能匹配 z 或 food。"(z|f)ood" 则匹配 zood 或 food。|
|[xyz]|字符集合。匹配所包含的任意一个字符。例如，[abc] 可以匹配 plain 中的 a。|
|[^xyz]|否定字符集合。匹配未包含的任意字符。例如，[^abc] 可以匹配 plain 中的 p。|
|[a-z]|字符范围。匹配指定范围内的任意字符。例如，[a-z] 可以匹配 a 到 z 范围内的任意小写字母字符。|
|[^a-z]|否定字符范围。匹配任何不在指定范围内的任意字符。例如，[^a-z] 可以匹配任何不在 a 到 z 范围内的任意字符。|
|\b|匹配一个单词边界，也就是指单词和空格间的位置。例如，er\b 可以匹配 never 中的 er ，但不能匹配 verb 中的 er。\b1_ 可以匹配1_23中的1_，但不能匹配21_3中的1_。|
|\B|匹配非单词边界。er\B 能匹配 verb 中的 er ，但不能匹配never 中的 er。|
|\cx|匹配由x指明的控制字符。例如，\cM 匹配一个Control-M 或 回车符。x 的值必须为A-Z或a-z之一。否则，将c视为一个原义的c字符。|
|\d|匹配一个数字字符。等价于 [0-9]。|
|\D|匹配一个非数字字符。等价于[^0-9]。|
|\f|匹配一个换页符。等价于\x0c和\cL。|
|\n|匹配一个换行符。等价于\x0a和\cJ。|
|\r|匹配一个回车符。等价于\x0d和\cM。|
|\t|匹配一个制表符。等价于\x09和\cI。|
|\v|匹配一个垂直制表符。等价于\x0b和\cK。|
|\s|匹配任何空白字符，包括空格、制表符、换页符等等。等价于[\f\n\r\t\v]。|
|\S|匹配任何非空白字符。等价于[^\f\n\r\t\v]。|
|\w|匹配包括下划线的任何单词字符。类似但不等价于[A-Za-z0-9_]，这里的单词字符使用Unicode字符集。|
|\W|匹配任何非单词字符。等价于[^A-Za-z0-9_]。|
|\xn|匹配n，其中n为十六进制转义值。十六进制转义值必须为确定的两个数字长。例如，\x41匹配A。\x041则等价于\x04&1。正則表达式中可以使用ASCII编码。|
|\num|匹配num，其中num是一个正整数。对所获取的匹配的引用。例如，(.)\1匹配两个连续的相同字符。|
|\n|标识一个八进制转义值或一个向后引用。如果\n之前至少 n 个获取的子表达式，则 n 为向后引用。否则，如果 n 为八进制数字（0-7），则 n 为一个八进制转义值。|
|\nm|标识一个八进制转义值或一个向后引用。如果\nm之前至少有 nm 个获得子表达式，则 nm 为向后引用。如果\nm 之前至少有 n 个获取，则 n 为一个后跟文字m的向后引用。如果前面的条件都不满足，若n和m均为八进制数字（0-7），则\nm将匹配八进制转义值nm。|
|\nml|如果n为八进制数字（0-7），且m和l均为八进制数字（0-7），则匹配八进制转义值nml。|
|\un|匹配n，其中n是一个用四个十六进制数字表示的Unicode字符。例如，\u00A9 匹配版权符号（©）。|

## 例子
1. 判断是不是 IP 白名单:
```python
#定义初始值
set $my_ip 0;

#判断是否为指定的白名单
if ( $http_x_forwarded_for ~* "10.0.0.1|172.16.0.1" ){
    set $my_ip 1;
}

#不是白名单的IP进行重定向跳转
if ( $my_ip = 0 ){
    rewrite ^/$ /40x.html;
}
```

2. 如果页面有多语言，但是这个多语言所在的文件找不到，那么就将多语言路径去掉，重新跳转
```python
location / {
    if (!-e $request_filename) {
        rewrite ^/([A-Za-z0-9_-]+)/(.*) /$2 permanent;
    }
}
```
比如你请求是这样子的 https://foo.com/zh-cn/a.html 这时候服务端找不到这个文件，那么就会重定向到 https://foo.com/a.html。 这个很适合那种有多语言静态页面的站点

3. 如果是一些特殊的静态文件，那么额外配置
```python
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
    root /webroot/res/;
}
```
4. 针对国内的搜索爬虫，返回中文的页面
```python
set $chinaspider "0";
if ($http_user_agent ~* "Baiduspider|Sogou spider|Sogou web spider") {
    set $chinaspider "1";
}
if ($chinaspider = 1) {
    return 301 https://$server_name/zh-cn;
}
```
5. 禁止以/data开头的文件
```python
location ~ ^/data {
  deny all;
 }
```

6. 文件反盗链并设置过期时间
 ```python
location ~* ^.+\.(jpg|jpeg|gif|png|swf|rar|zip|css|js)$ {
   valid_referers none blocked *.domain.com *.domain.net localhost 208.97.167.194;
    if ($invalid_referer) {
       rewrite ^/ http://error.domain.com/error.gif;
       return 412;
       break;
    }
    access_log  off;
    root /opt/lampp/htdocs/web;
    expires 3d;
    break;
 }
```
这里的 return 412 为自定义的 http 状态码，默认为403，方便找出正确的盗链的请求

+ rewrite ^/ http://error.domain.com/error.gif;显示一张防盗链图片
+ access_log off;不记录访问日志，减轻压力
+ expires 3d所有文件3天的浏览器缓存

## 最后附上可以用作判断的全局变量
|全局变量|内容|
|-|-|
|$remote_addr|获取客户端ip|
|$binary_remote_addr|客户端ip（二进制)|
|$remote_port|客户端port，如：50472|
|$remote_user|已经经过Auth Basic Module验证的用户名|
|$host|请求主机头字段，否则为服务器名称，如:blog.sakmon.com$request用户请求信息，如：GET ?a=1&b=2 HTTP/1.1|
|$request_filename|当前请求的文件的路径名，由root或alias和URI request组合而成，如：/2013/81.html|
|$status|请求的响应状态码,如:200|
|$body_bytes_sent|响应时送出的body字节数数量。即使连接中断，这个数据也是精确的,如：40|
|$content_length|等于请求行的Content_Length的值|
|$content_type|等于请求行的Content_Type的值|
|$http_referer|引用地址|
|$http_user_agent|客户端agent信息,如：Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/29.0.1547.76 Safari/537.36|
|$args|与 $query_string 相同 等于当中URL的参数(GET)，如a=1&b=2|
|$document_uri|与$uri相同, 这个变量指当前的请求URI，不包括任何参数(见$args) 如:/2013/81.html|
|$document_root|针对当前请求的根路径设置值|
|$hostname|如：centos53.localdomain|
|$http_cookie|客户端cookie信息|
|$cookie_COOKIEcookie| COOKIE变量的值|
|$is_args|如果有$args参数，这个变量等于 ”?”，否则等于”"|
|$limit_rate|这个变量可以限制连接速率，0 表示不限速|
|$query_string|与$args相同,等于当中URL的参数(GET)，如a=1&b=2|
|$request_body|记录POST过来的数据信息|
|$request_body_file|客户端请求主体信息的临时文件名|
|$request_method|客户端请求的动作，通常为GET或POST,如：GET|
|$request_uri|包含请求参数的原始URI，不包含主机名，如：/2013/81.html?a=1&b=2|
|$scheme|HTTP方法（如http，https）,如：http|
|$uri|这个变量指当前的请求URI，不包括任何参数(见$args) 如:/2013/81.html|
|$request_completion|如果请求结束，设置为OK。当请求未结束或如果该请求不是请求链串的最后一个时，为空(Empty)，如：OK|
|$server_protocol|请求使用的协议，通常是HTTP/1.0或HTTP/1.1，如：HTTP/1.1|
|$server_addr|服务器IP地址，在完成一次系统调用后可以确定这个值|
|$server_name|服务器名称，如：blog.sakmon.com|
|$server_port|请求到达服务器的端口号,如：80|

# 路由转发
# Rewrite语法规则

rewrite功能就是，使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。rewrite只能放在server{},location{},if{}中，并且只能对域名后边的除去传递的参数外的字符串起作用，例如http://seanlook.com/a/we/index.php?id=1&u=str 只对/a/we/index.php重写。

如果想修改域名或参数字符串，可以使用全局变量匹配，也可以使用proxy_pass反向代理。

rewrite和location功能有点像，都能实现跳转，主要区别在于rewrite是在同一域名内更改获取资源的路径，而location是对一类路径做控制访问或反向代理，可以proxy_pass到其他机器。

rewrite语法：`rewrite 正则表达式 要替换的内容 [flag];`

其中flag有如下几个值：

1. last – 本条规则匹配完成后，立即发起新一轮的location 匹配规则
2. break – 本条规则匹配完成即终止，不再匹配后面的任何规则
3. redirect – 返回302临时重定向，浏览器地址会显示跳转新的URL地址
4. permanent – 返回301永久重定向。浏览器地址会显示跳转新的URL地址

```shell
# rewrite 后面没有任何 flag 时就顺序执行 
# 当 location 中没有 rewrite 模块指令可被执行时 就重写发起新一轮location匹配
location / {
    # 顺序执行如下两条rewrite指令 
    rewrite ^/test1 /test2;
    rewrite ^/test2 /test3;  # 此处发起新一轮location匹配 uri为/test3
}

location = /test2 {
    return 200 "/test2";
}  

location = /test3 {
    return 200 "/test3";
}
# 发送如下请求
# curl 127.0.0.1:8080/test1
# /test3
```
**很多情况下rewrite也会写在location里，它们的执行顺序是：**

1. 顺序执行server块中的rewrite模块指令，得到rewrite后的请求URI
2. 执行location匹配 
3. 执行选定的location中的rewrite指令 
4. 如果其中某步URI被重写，则重新循环执行1-3，直到找到真实存在的文件；
5. 循环超过10次，则返回500 Internal Server Error错误。

## last 和 break的区别

1. last 和 break一样 它们都会终止此 location 中其他它rewrite模块指令的执行
2. 但是 last 立即发起新一轮的 location 匹配 而 break 则不会

```shell
location / {
    rewrite ^/test1 /test2;
    rewrite ^/test2 /test3 last;  # 此处发起新一轮location匹配 uri为/test3
    rewrite ^/test3 /test4;
    proxy_pass http://www.baidu.com;
}

location = /test2 {
    return 200 "/test2";
}  

location = /test3 {
    return 200 "/test3";
}
location = /test4 {
    return 200 "/test4";
}
# 发送如下请求
# curl 127.0.0.1:8080/test1
# /test3 

当如果将上面的 location / 改成如下代码
location / {
    rewrite ^/test1 /test2;
    # 此处 不会 发起新一轮location匹配；会终止执行后续rewrite 重写后的uri为 /more/index.html
    rewrite ^/test2 /more/index.html break;  
    rewrite /more/index\.html /test4; # 这条指令会被忽略

    # 因为 proxy_pass 不是rewrite模块的指令 所以它不会被 break终止
    proxy_pass https://www.baidu.com;
}
# 发送如下请求
# 浏览器输入 127.0.0.1:8080/test1 
# 代理到 百度产品大全页面 https://www.baidu.com/more/index.html;
```

## redirect 和 permanent

1. 临时重定向：对旧网址没有影响，但新网址不会有排名
2. 永久重定向：新网址完全继承旧网址，旧网址的排名等完全清零


修改nginx.conf文件：
```shell
server {
      listen 80 default;
      charset utf-8;
      server_name www.hzznb-xzll.xyz hzznb-xzll.xyz;

      # 临时（redirect）重定向配置 当访问 hzznb-xzll.xyz/temp_redir/ 这个请求会临时（302）重定向到百度页面
      location /temp_redir {
          rewrite ^/(.*) https://www.baidu.com redirect;
      }
      # 永久重定向（permanent）配置 当访问 hzznb-xzll.xyz/forever_red… 这个请求会永久（301）重定向到百度页面
      location /forever_redir {

          rewrite ^/(.*) https://www.baidu.com permanent;
      }

      # rewrite last配置 可以看到我们定义 访问 hzznb-xzll.xyz/1/ 的请求被替换为 hzznb-xzll.xyz/2/ 之后再被替换为 hzznb-xzll.xyz/3/  最后找到/usr/local/nginx/test/static/location_last_test.html 这个文件并返回。
      location /1 {
        rewrite /1/(.*) /2/$1 last;
      }
      location /2 {
        rewrite /2/(.*) /3/$1 last;
      }
      location /3 {
        alias  '/usr/local/nginx/test/static/';
        index location_last_test.html;
      }
    }

```

## rewrite 后的请求参数
如果替换字符串replacement包含新的请求参数，则在它们之后附加先前的请求参数。如果你不想要之前的参数，则在替换字符串 replacement 的末尾放置一个问号，避免附加它们。
```shell
# 由于最后加了个 ?，原来的请求参数将不会被追加到rewrite之后的url后面 
rewrite ^/users/(.*)$ /show?user=$1? last;
```

## rewrite_log
开启或者关闭 rewrite模块指令执行的日志，如果开启，则重写将记录下notice 等级的日志到nginx 的 error_log 中，默认为关闭 off

`Syntax: rewrite_log on | off;`

## return

格式：
```shell
return code [text];
return code URL;
return URL;
```
停止处理并将指定的code码返回给客户端。 非标准code码 444 关闭连接而不发送响应报头。

从0.8.42版本开始， return 语句可以指定重定向 url (状态码可以为如下几种 301,302,303,307),也可以为其他状态码指定响应的文本内容，并且重定向的url和响应的文本可以包含变量。

有一种特殊情况，就是重定向的url可以指定为此服务器本地的uri，这样的话，nginx会依据请求的协议$scheme， server_name_in_redirect 和 port_in_redirect自动生成完整的 url 

**此处要说明的是server_name_in_redirect和port_in_redirect 指令是表示是否将server块中的 server_name 和 listen 的端口 作为redirect用 ）**

```shell
# return code [text]; 返回 ok 给客户端
location = /ok {
    return 200 "ok";
}

# return code URL; 临时重定向到 百度
location = /redirect {
    return 302 http://www.baidu.com;
}

# return URL; 和上面一样 默认也是临时重定向
location = /redirect {
    return http://www.baidu.com;
}
```

## break
停止执行 ngx_http_rewrite_module 的指令集，但是其他模块指令是不受影响的

```shell
server {
    listen 8080;
    # 此处 break 会停止执行 server 块的 return 指令(return 指令属于rewrite模块)
    # 如果把它注释掉 则所有请求进来都返回 ok
    break;
    return 200 "ok";
    location = /testbreak {
        break;
        return 200 $request_uri;
        proxy_pass http://127.0.0.1:8080/other;
    }
    location / {
        return 200 $request_uri;
    }
}

# 发送请求如下
# curl 127.0.0.1:8080/testbreak
# /other

# 可以看到 返回 `/other` 而不是 `/testbreak`，说明 `proxy_pass` 指令还是被执行了
# 也就是说 其他模块的指令是不会被 break 中断执行的
# (proxy_pass是ngx_http_proxy_module的指令)

```