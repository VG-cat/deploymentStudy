# nginx

```
apt-get update

apt install nginx

nginx -v  查看版本信息

service nginx stop
service nginx start
```



cd ~ 切换到当前用户的主目录（Home Directory）。

cd / 切换到系统的根目录（Root Directory）。

## 相关目录

```
cd /etc/nginx/    配置信息目录

cd /var/log/nginx/   日志目录

cd /usr/share/nginx/html    默认页面，首页界面
```

```
#  /etc/nginx/

conf.d      自定义配置文件夹 
nginx.conf  全局配置文件
```

## 参数解释

```
user www-data;          用户名
worker_processes auto;   进程数量
pid /run/nginx.pid;      进程号
include /etc/nginx/modules-enabled/*.conf;

events {  
	worker_connections 768;     # 单进程最大连接数
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;    开启高效文件传输模式
	autoindex on;   开启文件列表访问，适合下载服务器
	tcp_nopush on;  防止网络阻塞
	tcp_nodelay on; 防止网络阻塞
	keepalive_timeout 65;   长连接超时时间
	types_hash_max_size 2048;  
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;   处理的文件类型
	default_type application/octet-stream; 默认文件类型

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;  开启gzip压缩输出

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
	
	server {
		# 匹配路由，返回指定目录下的指定页面
		listen 10.1.2.1:110;
		location / {
			root /etc/nginx/html;
			index index.html;
		}
	}
}


#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
# 
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
# 
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
# 
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}

```

```
server {
		# 匹配路由，返回指定目录下的指定页面
		
		# listen 只写端口，表示0.0.0.0：端口
		listen 10.1.2.1:110; 
		
		#匹配规则
		location / {
			# 指定文件目录
			root /etc/nginx/html;
			# 执行首页文件
			index index.html;
		}
		# 优先级：精确匹配 > 优先匹配 > 通用匹配
		#通用匹配
		location / {
			return 400
		}
		#精确匹配
		location = / {
			return 400
		}
		#优先匹配
		location ~ / {
			return 400
		}
		
	}
```



```
# try_files和return的区别: try_files对于根路径匹配无效

location /a {
	try_files $uri  $uri/ =400
}

# 配合@使用
location /a {
	try_files $uri  $uri/ @meiduo
}
location @meiduo {
	return 402
}
```

```
# return 的跳转功能(重定向)

location / { 
	return 302 https://www.baidu.com
}
```

```
# root和alias的区别

根路径用root;
location / {			
	root /etc/nginx/html;			
	index index.html;
}
子路径用alias;
location / {			
	alias /etc/nginx/html;			
	index index.html;
}
```

```
#访问控制
location / {
	allow 10.1.1.30   # 允许的客户端
	deny all;    # 除了允许，拒绝全部
	root /etc/nginx/html;			
	index index.html;
}
```

```
#文件下载目录
location /down {
	alias /etc/nginx/html;		# 指定下载目录
	autoindex on;        # 开启下载
}
```



## 重新导入

修改完配置文件后

```
nginx -t  # 测试语法
nginx -s reload
```

## 开启服务器ping功能

- 如果想打开ICMP协议：

```
echo “1”>/proc/sys/net/ipv4/icmp_echo_ignore_all
cat /proc/sys/net/ipv4/icmp_echo_ignore_all
```

## 反向代理

正向代理：代理客户端，代理服务器隐藏客户端信息

反向代理：代理服务端，代理服务器隐藏服务端信息

```
server {
	listen 10.1.2.1:80; 
	location / {
		proxy_pass http://10.1.2.1:8000;   # 转发到8000
	}
}

server {
	listen 10.1.2.1:8000; 
	location / {
		root /etc/nginx/html;
	}
}
```

## 负载均衡

当有多个web服务时，需要负载均衡

```
upstream meiduo {
	# 默认轮询算法
	#server 10.1.1.1:8000;
	#server 10.1.1.1:8001;
	#server 10.1.1.1:8002;
	
	#加权
	server 10.1.1.1:8000 weight=1;
	server 10.1.1.1:8001 weight=2;
	server 10.1.1.1:8002 weight=3;
	
	#ip_hash算法
	ip_hash;
}

server {
	listen 10.1.2.1:80; 
	location / {
		proxy_pass http://meiduo;   
	}
}

server {
	listen 10.1.2.1:8000; 
	location / {
		root /etc/nginx/html;
	}
}
....
```



## 日志处理

可以修改输出的日志样式

```
nginx.conf

log_format proxy_format '$remote_add  $http_x_real_ip $http_x_forwarded_for'
```

```
其他配置文件

server {
	listen 10.1.2.1:80; 
	location / {
		proxy_pass http://meiduo;   
		proxy_set_header X-Real-IP $remote_addr;      #添加变量，防止代理之后无法找到原ip
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
}

server {
	listen 10.1.2.1:8000; 
	
	access_log /var/log/nginx/app/access.log proxy_format;  #按照指定模式，保存日志
	real_ip_header X-Forwarded-For;
	real_ip_recursive on;  #开启获取真实ip
	
	location / {
		root /etc/nginx/html;
	}
}
```

