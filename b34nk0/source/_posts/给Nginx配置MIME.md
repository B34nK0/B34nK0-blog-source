---
title: 给Nginx配置MIME
date: 2022-03-08 17:09:16
categories: 前端
tags: 问题
---

打包前端项目部署时，运动前端无法加载样式的问题。原因是浏览器没有识别返回的css等文件。

解决方式：
在nginx给http配置MIME类型映射


nginx目录:/usr/local/nginx/


在顶级目录下添加文件mime.types

```
    types {
        text/html                                        html htm shtml;
        text/css                                         css;
        text/xml                                         xml;
        text/csv                                         csv;
        image/gif                                        gif;
        image/jpeg                                       jpeg jpg;
        application/javascript                           js;
        application/atom+xml                             atom;
        application/rss+xml                              rss;

        text/mathml                                      mml;
        text/plain                                       txt;
        text/vnd.sun.j2me.app-descriptor                 jad;
        text/vnd.wap.wml                                 wml;
        text/x-component                                 htc;

        image/png                                        png;
        image/svg+xml                                    svg svgz;
        image/tiff                                       tif tiff;
        image/vnd.wap.wbmp                               wbmp;
        image/webp                                       webp;
        image/x-icon                                     ico;
        image/x-jng                                      jng;
        image/x-ms-bmp                                   bmp;

        font/woff                                        woff;
        font/woff2                                       woff2;

        application/java-archive                         jar war ear;
        application/json                                 json;
        application/mac-binhex40                         hqx;
        application/msword                               doc;
        application/pdf                                  pdf;
        application/postscript                           ps eps ai;
        application/rtf                                  rtf;
        application/vnd.apple.mpegurl                    m3u8;
        application/vnd.google-earth.kml+xml             kml;
        application/vnd.google-earth.kmz                 kmz;
        application/vnd.ms-excel                         xls;
        application/vnd.ms-fontobject                    eot;
        application/vnd.ms-powerpoint                    ppt;
        application/vnd.oasis.opendocument.graphics      odg;
        application/vnd.oasis.opendocument.presentation  odp;
        application/vnd.oasis.opendocument.spreadsheet   ods;
        application/vnd.oasis.opendocument.text          odt;
        application/vnd.openxmlformats-officedocument.presentationml.presentation
                                     pptx;
        application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
                                     xlsx;
        application/vnd.openxmlformats-officedocument.wordprocessingml.document
                                     docx;
        application/vnd.wap.wmlc                         wmlc;
        application/x-7z-compressed                      7z;
        application/x-cocoa                              cco;
        application/x-java-archive-diff                  jardiff;
        application/x-java-jnlp-file                     jnlp;
        application/x-makeself                           run;
        application/x-perl                               pl pm;
        application/x-pilot                              prc pdb;
        application/x-rar-compressed                     rar;
        application/x-redhat-package-manager             rpm;
        application/x-sea                                sea;
        application/x-shockwave-flash                    swf;
        application/x-stuffit                            sit;
        application/x-tcl                                tcl tk;
        application/x-x509-ca-cert                       der pem crt;
        application/x-xpinstall                          xpi;
        application/xhtml+xml                            xhtml;
        application/xspf+xml                             xspf;
        application/zip                                  zip;

        application/octet-stream                         bin exe dll;
        application/octet-stream                         deb;
        application/octet-stream                         dmg;
        application/octet-stream                         iso img;
        application/octet-stream                         msi msp msm;

        audio/midi                                       mid midi kar;
        audio/mpeg                                       mp3;
        audio/ogg                                        ogg;
        audio/x-m4a                                      m4a;
        audio/x-realaudio                                ra;

        video/3gpp                                       3gpp 3gp;
        video/mp2t                                       ts;
        video/mp4                                        mp4;
        video/mpeg                                       mpeg mpg;
        video/quicktime                                  mov;
        video/webm                                       webm;
        video/x-flv                                      flv;
        video/x-m4v                                      m4v;
        video/x-mng                                      mng;
        video/x-ms-asf                                   asx asf;
        video/x-ms-wmv                                   wmv;
        video/x-msvideo                                  avi;
    }
···

在nginx配置文件nginx.conf里给http配置mime.types加载

```

http {
    log_format upstreamlog '[$time_local] $remote_addr - $remote_user - $server_name $host to: $upstream_addr: $request $status upstream_response_time $upstream_response_time msec $msec request_time $request_time';
    #access_log  /dev/stdout upstreamlog ;
    
    etag off;
    server_tokens   off;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   10;
    types_hash_max_size 2048;
    reset_timedout_connection on;
    client_body_timeout 30;
    send_timeout 5;
    client_header_buffer_size 4k;
    open_file_cache max=65535 inactive=60s;
    
    #mime.types路径为nginx顶级目录下
    include             mime.types;
}
```

查看前端http请求后的返回Content-Type