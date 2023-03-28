# Nginx

## [nginx.conf]
```nginx
worker_processes auto;
events {
    worker_connections 1024;
}

http {
    include mime.types;
    default_type application/octet-stream;
    sendfile on;
    keepalive_timeout 65;

    # 프락시 (라운드로빈 방식) 설정 (1/2)
    # [참조](...)
    upstream backendserver {
        server 127.0.0.1:1111;
        server 127.0.0.1:2222;
        server 127.0.0.1:3333;
        server 127.0.0.1:4444;
    }

    server {
        listen 8080;
        server_name localhost;
        charset utf-8;

        # 서버루트 (root path) 경로
        root /Users/vivabm/Web/1_Public/root;

        # Rewrte Sample
        rewrite ^/number/(\w+) /count/$1;

        # 프락시 (라운드로빈 방식) 설정 (2/2)
        location / {
            proxy_pass http://backendserver;
        }

        # Range Sample
        location ~* /count/[0-9] {
            root /Users/vivabm/Web/1_Public/root;
            try_files /index.html =404;
        }

        # Sub Directory Sample
        location /csharp {
            root /Users/vivabm/Web/1_Public/root;
            index index.html;
        }

        # Alias to csharp Sample
        location /blazor {
            alias /Users/vivabm/Web/1_Public/root/csharp;
        }

        # default page file not exist default page view example
        location /python {
            root /Users/vivabm/Web/1_Public/root;
            try_files /vegetables/python.html /index.html =404;
        }

        # if asp-net redirect to csharp
        location /asp-net {
            return 307 /csharp;
            # -> http://localhost:8080/csharp/
        }

        # each sampe name page examples
        # (1) about
        location /about {
            root /Users/vivabm/Web/1_Public/root;
            index about.html;
        }
        # (2) company
        location /company {
            root /Users/vivabm/Web/1_Public/root;
            index company.html;
        }

        # (3) contents
        location /contents {
            root /Users/vivabm/Web/1_Public/root;
            index contents.html;
        }
    }
    include servers/*;
}
```

# 연관 데이터
## [Dockerfile Example](https://github.com/ViVaKR/Root/blob/main/server/Dockerfile) ##
> Round-Robin Proxy Settings Example
