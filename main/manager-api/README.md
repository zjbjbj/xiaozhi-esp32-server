本文档是开发类文档，如需部署小智服务端，[点击这里查看部署教程](../../README.md#%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3)

# 项目介绍

manager-api 该项目基于SpringBoot框架开发。

开发使用代码编辑器，导入项目时，选择`manager-api`文件夹作为项目目录

# 开发环境
JDK 21
Maven 3.8+
MySQL 8.0+
Redis 5.0+
Vue 3.x

# 接口文档
启动后打开：http://localhost:8002/xiaozhi/doc.html


# nginx反向代理manager-api和manager-web教程
user  root;
worker_processes  4;

events {
worker_connections  1024;
}

http {
include       mime.types;
default_type  application/octet-stream;
sendfile        on;
keepalive_timeout  300;
client_header_timeout 180s;
client_body_timeout 180s;
client_max_body_size 1024M;

    gzip on;
    gzip_buffers 32 4K;
    gzip_comp_level 6;
    gzip_min_length 100;
    gzip_types application/javascript text/css text/xml image/jpeg image/gif image/png;
    gzip_disable "MSIE [1-6]\.";
    gzip_vary on;

    server {
        # 无域名访问，就用localhost
        server_name localhost;
        # 80端口
        listen  80;

        # 把前端编译后的页面当作首页
        location / {
            # 请注意，这是manager-web编译后的路径，不是manager-web源码路径
            root   /home/system/xiaozhi/manager-web/;
            index  index.html;
        }

       # 把移动端h5
        location /h5/ {
            # 请注意，由于我把manager-mobile编译后产生的h5文件夹放在/home/system/xiaozhi/下，所以就是这么写
            root   /home/system/xiaozhi/;
            index  index.html;
        }

        location /test/ {
            root   /home/system/xiaozhi/xiaozhi-esp32-server/main/xiaozhi-server/;
            index  test_page.html;
        }
        # API反向代理（manager-api 8002项目）
        location /xiaozhi/ {
            proxy_pass http://127.0.0.1:8002;
            proxy_set_header   Host   $host;
            proxy_cookie_path /fabric/ /;
            proxy_set_header   Referer $http_referer;
            proxy_set_header   Cookie $http_cookie;

            proxy_connect_timeout 15;
            proxy_send_timeout 15;
            proxy_read_timeout 15;

            proxy_set_header   X-Real-IP  $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        # API反向代理（Python 8000项目）
        location /xiaozhi/v1/ {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_cookie_path /xiaozhi/ /;

            # 核心IP转发配置
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            # WebSocket支持（保持原有配置）
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";

            # 其他头信息（可选）
            proxy_set_header Referer $http_referer;
            proxy_set_header Cookie $http_cookie;
        }
        # 拍照视觉等MCP
        location /mcp/ {
            proxy_pass http://127.0.0.1:8003;
            proxy_set_header   Host   $host;
            proxy_cookie_path /mcp/ /;
            proxy_set_header   Referer $http_referer;
            proxy_set_header   Cookie $http_cookie;

            proxy_connect_timeout 10;
            proxy_send_timeout 10;
            proxy_read_timeout 10;

            proxy_set_header   X-Real-IP  $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}

# 把移动端h5
location /h5/ {
# 请注意，由于我把manager-mobile编译后产生的h5文件夹放在/home/system/xiaozhi/下，所以就是这么写
root   /home/system/xiaozhi/;
index  index.html;
}

VITE_APP_PUBLIC_BASE=/h5/