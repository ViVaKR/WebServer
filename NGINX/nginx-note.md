# nginx

## 마스터와 워커프로세스 로 구성

> **client** -_(요청)_-> **master** -_(전달)_-> **worker** -> _(처리)_

## Configuration

+ nginx -s _signal_
  + stop
  + quit
  + reload
  + reopen

## 프로세스 확인

```base
    ps -ax | grep nginx
```

## 정적 콘텐츠 제공

```nginx
    http {
        server {
            
            location / {
                root /data/www;
            }
            location /images {
                root /data;
            }
        }
    }
```

## 단순 프록시 서버 설정

```nginx
    server {
        listen 8080;
        root /data/up1;
        
        location / {
            
        }
    }
    
    server {
        location / {
            proxy_pass http://localhost:8080;
        }
    }
```

## 로드밸런싱

+ 기본 알고리즘 : `라운드로빈`  
+ 가중치
  + least_conn : 최소연결, 최소 활성 연결 수로 서버 가중치 다시 계산  
  + ip_hash : 클라이언트 IP 주소에서 결정하며 동일한 서버로 전달
  + weight : 서버 가중치 -> `weight=5`  
  + slow_start : 느린시작 -> `slow_start=30s`  
  + max_conns : 연결 수 제한 -> `max_conn=3`

```nginx

# 백 엔드 그룹정의
http {
    upstream backend {
        server backend1.example.com weight=5;
        server backend2.example.com;
        server 192.0.0.1 backup;
    }
}

# 요청 전달
server {
    location / {
        proxy_pass http://backend;
    }
}

```

>> 서버제거 : 매개 변수 -> `down` 키워드  

```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com down;
}
```

## [nginx.conf] examples

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

## [Dockerfile Example](https://github.com/ViVaKR/Root/blob/main/server/Dockerfile)

> Round-Robin Proxy Settings Example

## Docroot : `/usr/local/var/www`

## Load Files : `/usr/local/etc/nginx/serviers/.`

## ETC Commands

```bash

    # Restart nginx
    $ brew services restart nginx
    
    # turn off service 
    $ /usr/local/opt/nginx/bin/nginx -g daemon off;

    $ nginx -v # 버전정보
    $ nginx -t # 구문체크
    $ nginx -s reload # 시그컬 (stop, quit, reload, reopen) 등의 시그널 전달
```

## ========== (nginx.conf) ==========

+ Directives
  + user
  + error_log
  + worker_processes
  + Top-Level directives
    + events : General connection processing
    + http : HTTP Traffic
    + mail : Mail Traffic
    + stream : TCP and UDP traffi
+ NGINX Module Variables
  + (ref) '<http://nginx.org/en/docs/varindex.html>'
  + (ex) <https://domain.com:8080/prod/get?id=32>
  + $sheme    => https
  + $host     => domain.com
  + $uri      => /prod/get
  + $args     => 32
  + $server_addr  => 서버주소
  + $server_name  => 서버이름
  + $server_port  => 서버포트
  + $server_protocol  => HTTP 요청프로토콜 (HTTP/1.0 | HTTP/1.1)
  + $cookie_COOKIE    => 쿠키값
+ Confuguration Variable (사용자 변수)
+ e.g. set $var 'values';
+ 패턴정의 문자
  + ^ : 일치시킬 문자열의 시작을 나타냄
  + $ : 일치시킬 문자열의 끝을 나타냄
  + ? : 일치항목이 발견되면 패턴 검색을 중지
