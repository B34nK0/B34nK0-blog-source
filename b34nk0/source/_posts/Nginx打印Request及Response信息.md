---
title: Nginx打印Request及Response信息
date: 2022-03-09 11:16:48
categories: 运维
tags: Nginx
---

场景：
生产过程中，使用Nginx做反向代理转发请求到第三方网关时，因为请求出错，所以需要在nginx 打印请求的信息。

# 配置Nginx 日志格式

使用lua脚本需要nginx配置了lua，或者使用openresty取代nginx
```
http {
    #默认值为off，忽略变量下划线
    underscores_in_headers on;
    
    #设置日志格式，采用$http_xxx的方式记录header name(xxx)的值
    log_format upstreamlog '$remote_addr - $remote_user [$time_local] '
        '"$request" $status $body_bytes_sent '
        '"$http_referer" "$http_user_agent" $request_time req_body:"$request_body" authorization_for_digquant:"$http_authorization_for_digquant"  Cookie:"$http_Cookie"';

    access_log logs/access.log  upstreamlog ;

    #使用lua方式输出resp
    set $resp_header "";
    header_filter_by_lua '
        local h = ngx.resp.get_headers()
        for k, v in pairs(h) do
        ngx.var.resp_header=ngx.var.resp_header..k..": "..v
        end
    ';

    set $resp_body "";
    body_filter_by_lua '
        local resp_body = string.sub(ngx.arg[1], 1, 1000)
        ngx.ctx.buffered = (ngx.ctx.buffered or "") .. resp_body
        if ngx.arg[2] then
            ngx.var.resp_body = ngx.ctx.buffered
        end
    ';
   
}

```

# 问题

在请求过程中，还遇到一个问题，在header中设置了字段信息后，Nginx的log始终读取不到值。

原因在于header name使用了下划线，nginx对header name的字符做了限制，默认 underscores_in_headers 为off，表示如果header name中包含下划线，则忽略掉。

# 解决
1、在header里不要用 “_” 下划线，可以用驼峰命名或者其他的符号（如减号-）代替。nginx默认忽略掉下划线可能有些原因。

2、在nginx里的 nginx.conf文件中配置http的部分添加`underscores_in_headers on;`（默认值是off）

因为是第三方定义的属性名，所以采用第二种方案，设置`underscores_in_headers`值为`on`;