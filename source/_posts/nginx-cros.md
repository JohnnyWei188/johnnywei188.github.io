---
title: 记录一次解决nginx跨域的问题 
date: 2018-08-13
tags:
  - nginx 
categories:
  - 技术
---

##### CROS是什么？ 
CROS,全称是跨域资源共享 (Cross-origin resource sharing)，它的提出就是为了解决跨域请求的。

#### nginx中的一些配置
```
add_header 'Access-Control-Allow-Origin' 'http://demo.csrf.com';
add_header 'Access-Control-Allow-Methods' 'GET,POST,OPTIONS';
add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
add_header 'Access-Control-Expose-Headers' 'X-Custom-Header';

if ($request_method = 'OPTIONS') {
    return 204;
}
```

#### 遇到的一些问题
- 设置Access-Control-Allow-Origin的时候, 仅仅只写了demo.csrf.com，没有带上http，所有，域名设置有问题
- 因为客户端请求的时候，有带入一些自定义的头，X-Custom-Header，如果nginx里面如果没有配置Access-Control-Expose-Headers这个参数的话，客户端请求的时候也会报错
- 其他需要注意的就是Methods，必须包含OPTIONS。为什么呢？ 因为，浏览器在发送请求的时候，有分2种情况: 
- - 第一种: 简单请求。简单请求必须满足2个条件：1) 请求方法只能是HEAD、GET、POST中的一种 2) http header中不能超出一下几种字段: Accept、Accept-language、Content-Language、Last-Event-ID、Content-Type只能包含3种值application/x-www-form-urlencoded、multipart/form-data、text/plain
- - 第二种: 复杂的请求。就是不满足上面简单请求条件的, 比如，有自定义的http头，或者Content-Type为 json/application的这种，当浏览器发现，这个请求为一个复杂请求的时候，就会发送一个OPTIONS的http method进行探测, 所以，这就解释了为什么必须包含OPTIONS这个方法。在探测的时候，如果服务返回了对应的Access-Control-Expose-Headers、Access-Control-Allow-Origin等，与客户端请求的能够匹配得上，则浏览器判定为整个是一个安全的请求，进行进行真实的POST等请求
