---
title: nginx rtmp 安装篇
date: 2017-03-28 16:55:00
tags: 直播
---

## nginx
（发音同engine x）是一个网页服务器，它能反向代理HTTP, HTTPS, SMTP, POP3, IMAP的协议链接，以及一个负载均衡器和一个HTTP缓存。

### nginx rtmp安装教程
通过神器homebrew安装

- brew update 更新本地brew
- brew install homebrew/nginx/nginx-full --with-rtmp-module<br>nginx-full代码nginx的完整版
- 查看安装后的信息brew info nginx-full

### 修改nginx的配置文件
>通过以上nginx后的配置文件路径在/usr/local/etc/nginx下
修改配置文件为以下（去掉了注释部分）<br>

		#user  nobody;
		worker_processes  1;

		error_log  logs/error.log debug;

		events {
    		worker_connections  1024;
		}

		http {
    		include       mime.types;
    		default_type  application/octet-stream;

    		sendfile        on;
    		keepalive_timeout  65;

    		server {
		        listen       8080;
		        server_name  localhost;
		
		        # rtmp stat
		        location /stat {
		            rtmp_stat all;
		            rtmp_stat_stylesheet stat.xsl;
		        }
		        location /stat.xsl {
		            # 这个路径一定要改
		            root /usr/local/Cellar/rtmp-nginx-module/1.1.7.9/share/rtmp-nginx-module;
		        }

		        # rtmp control
		        location /control {
		            rtmp_control all;
		        }
		
		        error_page   500 502 503 504  /50x.html;
		        location = /50x.html {
		            root   html;
		        }
    		}
	}

	rtmp {
	    server {
	        listen 1935;
	        ping 30s;
	        notify_method get;
	
	        application myapp {
	            live on;
	        }
	    }
	}

 编辑之后通过sudo nginx -s reload重启nginx。
 启动方式，进入ngix的目录，如我安装的目录：/usr/local/Cellar/nginx-full/1.10.3/bin 启动 sudo ./nginx
 
## ffmpeg安装
### ffmpeg安装教程 
1. brew unlink ffmpeg
2. brew install ffmpeg --with-ffplay

1. 访问 http://localhost:8080/stat，如果正常打开，表示你已经成功了一大半
2. cd /System/Library/Compositions/
3. ffmpeg -re -i Yosemite.mov -c copy -f flv rtmp://localhost/myapp/mystream
4. ffplay rtmp://localhost/myapp/mystream
开始可能会出现如下的连接错误
Connection to tcp://localhost:1935 failed (Connection refused), trying next address

