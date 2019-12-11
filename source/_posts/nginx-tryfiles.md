---
title: nginx中的tryfiles命令 
date: 2019-12-11
tags:
  - nginx 
categories:
  - 技术
---

##### nginx的tryfiles命令 
今天遇到了一个路径需要重写的问题，发现nginx除了可以使用rewrite命令，也可以使用tryfiles命令
<!--more-->

#### tryfiles和rewrite之间有什么区别呢？ 找了一下官方手册，目前官网中，推荐配置也是tryfiles
```
Example for Drupal/FastCGI:

location / {
    try_files $uri $uri/ @drupal;
}

location ~ \.php$ {
    try_files $uri @drupal;

    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME     $fastcgi_script_name;
    fastcgi_param QUERY_STRING    $args;

    ... other fastcgi_param's
}

location @drupal {
    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to/index.php;
    fastcgi_param SCRIPT_NAME     /index.php;
    fastcgi_param QUERY_STRING    q=$uri&$args;

    ... other fastcgi_param's
}
```
找了一下文档，原来是在fpm中，如果使用rewrite的时候会把所有的文件都重定向然后发送给php-fpm包括了静态文件, 而使用tryfiles可以提供效率


#### 我自己在本地试了下tryfiles命令，记录一下
```
    location  /appeal/ {
        #try_files /index.html index.html =404;
        #try_files $uri /appeal/1.html =404;        # 正常
        #try_files $uri /tryfiles/index.html =404;  # 正常
        #try_files $uri /index.html =404;           # 正常
        try_files $uri /test/$uri =404;
    }
```
需要注意的是，这里的uri 是一整个路径如上面，如果请求的地址为: http://localhost.com/appeal/index.html, 而appeal下木有index.html，那么，uri的值为appeal/index.html，其实从uri字面理解，也
应该是一个uri地址，而不是一个匹配到的文件。我一开始没注意，总以为是匹配上的那个文件index.html。
我的目的是希望请求appeal目录下的index.html文件，如果没有，则请求/目录下的，所以这样的配置一直都是404：
``` 
location /appeal/ {
	try_files $uri /$uri =404;
}
```


