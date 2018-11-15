soa 是...

soa里面还有一个xlsoa.core.certificate, 这个服务主要用来实现服务的鉴权，授权管理等

consul 用于服务注册以及发现,即我们所有的服务都注册到consul上去

envoy 是一个使用C++编写的高可用的proxy  可以用于流量控制，熔断等

eds是我们在使用的时候针对envoy某些接口的封装
