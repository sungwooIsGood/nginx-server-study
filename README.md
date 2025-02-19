## Nginx 설치 및 실행 (Ubuntu)

**Ubuntu 환경에서 최신 버전의 Nginx를 설치, 실행 및 기초 학습을 위한 공간**

일반적으로 `apt install nginx` 명령어로 설치할 수도 있지만, **최신 버전을 설치하기 위해 추가 패키지 작업을 거쳤음.**

### Nginx 필수 패키지 설치

먼저, Nginx 설치에 필요한 기본 패키지와 Nginx를 설치해보자.

```bash
# 1️⃣ 필수 패키지 설치
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring

# 2️⃣ 공식 GPG 키 가져오기 (패키지 인증을 위한 것)
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null    

# 3️⃣ 저장소 추가 (Nginx 최신 버전이 있는 곳)
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list

# 4️⃣ 저장소 우선순위 설정 (기본 Ubuntu 저장소보다 Nginx 공식 저장소가 우선)
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx

# 5️⃣ 패키지 목록 업데이트 후 설치
sudo apt update
sudo apt install nginx
```

**참고:**

- **최신 버전의 Nginx를 설치하려면?** → 위 과정을 따르면 된다.
- **안정적인 버전만 필요하다면?** → 아래 명령어로 설치하면 된다.

```bash
sudo apt update
sudo apt install nginx
```

### Nginx 실행 및 상태 확인

설치가 완료 되었으면, Nginx를 실행 및 상태 확인

```bash
# Nginx 상태 확인
sudo systemctl status nginx

# Nginx 실행
sudo systemctl start nginx

```

### Nginx 로그 확인

Nginx 로그 파일은 `/var/log/nginx/` 경로에 저장된다.

```bash
cd /var/log/nginx
ls  # access.log, error.log 확인

# 최근 로그 출력
tail access.log

# 실시간 로그 모니터링
tail -f access.log

```
---

## Nginx 설정 파일 구조 및 기본 설정

### Nginx 설정 파일 위치

Nginx의 설정 파일은 `/etc/nginx` 경로에 위치하며, 기본적인 설정 파일은 다음과 같다.

```bash
cd /etc/nginx
```

```bash
conf.d          # 개별 서버 설정 파일 폴더
fastcgi_params  # FastCGI 관련 설정
mime.types      # MIME 타입 설정
modules         # Nginx 모듈
nginx.conf      # Nginx 전반적인 설정 파일
scgi_params     # SCGI 관련 설정
uwsgi_params    # uWSGI 관련 설정
```

### nginx.conf (Nginx 전역 설정 파일)

`nginx.conf`는 Nginx의 전반적인 동작을 설정하는 파일로, 주요 내용을 살펴보면 다음과 같다. 기초 단계에서는 당장 nginx.conf 파일에서 설정 할 것은 없다. 즉, 커스텀 설정파일을 `include`를 통해 링크해주면 된다. `/etc/nginx/conf.d/*.conf;` 개별 서버 설정 파일 포함하는 문법이다.

```
user  nginx;  # Nginx 프로세스를 실행할 사용자
worker_processes  auto;  # 사용 가능한 CPU 코어 수에 따라 worker 프로세스 자동 설정

error_log  /var/log/nginx/error.log notice;  # 에러 로그 위치
pid        /var/run/nginx.pid;  # Nginx 프로세스 ID 파일

events {
    worker_connections  1024;  # 하나의 worker 프로세스가 처리할 수 있는 최대 연결 수
}

http {
    include       /etc/nginx/mime.types;  # MIME 타입 설정 파일 포함
    default_type  application/octet-stream;  # 기본 MIME 타입

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;  # 접근 로그 설정

    sendfile        on;  # 파일 전송 최적화 활성화
    keepalive_timeout  65;  # Keep-Alive 연결 유지 시간 설정

    include /etc/nginx/conf.d/*.conf;  # 개별 서버 설정 파일 포함
}
```

### conf.d/default.conf (서버 블록 설정)

기본적으로 `conf.d` 폴더 안에 있는 `default.conf` 파일에서 개별 서버의 동작을 설정할 수 있다.

```bash
cd /etc/nginx/conf.d
```

기본 설정 내용:

```

server {
    listen       80;  # 서버가 80번 포트에서 HTTP 요청 수신
    server_name  localhost;  # 서버 이름 설정

    location / {
        root   /usr/share/nginx/html;  # 웹 루트 디렉토리 설정
        index  index.html index.htm;  # 기본 제공할 파일 목록
    }

    error_page   500 502 503 504  /50x.html;  # 에러 발생 시 표시할 페이지 설정
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # PHP 파일 처리를 위한 FastCGI 설정 예시 (주석 처리됨)
    # location ~ \.php$ {
    #     fastcgi_pass   127.0.0.1:9000;
    #     fastcgi_index  index.php;
    #     fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #     include        fastcgi_params;
    # }

    # .htaccess 접근 차단 (Apache의 .htaccess 파일 보안 설정)
    # location ~ /\.ht {
    #     deny  all;
    # }
}

```

### 주요 설정 설명

| 설정 | 설명 |
| --- | --- |
| `server` | 하나의 웹 사이트에 관련된 설정을 관리하는 단위 (server 블록이라고한다.) |
| `listen 80;` | 80번 포트에서 HTTP 요청을 수신 |
| `server_name localhost;` | 서버의 도메인 또는 호스트명 설정, 만약 ip로 치고 들어 왔을 경우 -> server_name에 일치하는 블럭이 없을 경우 첫 번째 정의되어 있는 server 블럭 기반으로 처리한다. |
| `location / {}` | /로 시작하는 모든 경로를 처리 |
| `root /usr/share/nginx/html;` | 웹사이트 루트 디렉토리 설정 |
| `index index.html index.htm;` | 기본 파일 설정 (index.html 또는 index.htm) |
| `error_page 500 502 503 504 /50x.html;` | 서버 오류 발생 시 보여줄 페이지 지정 |
| `location = /50x.html` | /50x.html과 완전히 일치하는 경로를 처리하며, {} 블록안에 있는 /usr/share/nginx/html 폴더 안에 있는 /50x.html 파일로 처리하겠다는 의미이다.  |
| `location ~ \.php$ { ... }` | PHP 요청을 FastCGI 서버로 전달 (현재 주석 처리됨) |
| `location ~ /\.ht { deny all; }` | `.htaccess` 파일 접근 차단 |

### 설정 적용 방법

Nginx 설정을 변경한 후 적용하려면 아래 명령어를 실행한다.

```bash
# 설정 파일 문법 확인
nginx -t

# Nginx 재시작
systemctl restart nginx
```

---

## 🛠️ Nginx 디버깅 방법

Nginx에서 설정 변경 후 문제가 발생하거나, 특정 경로 및 파일이 정상적으로 동작하지 않을 때 디버깅하는 방법을 정리합니다.

---

### 1️. Nginx 서비스 상태 확인

먼저 Nginx가 실행 중인지 확인한다.

```bash
sudo systemctl status nginx
```

- `active (running)` 상태라면 정상 실행 중
- `inactive (dead)` 또는 `failed` 상태라면 실행되지 않고 있는 것

만약 실행되지 않는다면 아래 명령어로 다시 실행할 수 있다.

```bash
sudo systemctl restart nginx
```

### 2️. Nginx 설정 문법 검사

Nginx 설정 파일의 문법 오류를 확인하는 명령어이다.

```bash
sudo nginx -t

```

- 성공 예시:
```bash
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

- 실패 예시:
```bash
nginx: [emerg] unknown directive "serverx" in /etc/nginx/conf.d/default.conf:2
nginx: configuration file /etc/nginx/nginx.conf test failed
```

---

### 3️. Nginx 로그 확인

설정 문법에는 문제가 없지만 파일 경로나 설정이 잘못되었을 수 있다.
이때, Nginx 로그를 확인하는 것이 중요하다.

### Nginx 로그 디렉토리 이동

```bash
cd /var/log/nginx
ls -l
```

로그 파일 목록이 출력된다.

```bash
-rw-r--r-- 1 root root  123456 Feb 19 10:00 access.log
-rw-r--r-- 1 root root  654321 Feb 19 10:00 error.log

```

### Access Log 확인 (요청이 Nginx까지 도달했는지 확인)

```bash
sudo tail -f access.log
```

- 요청이 정상적으로 들어온 경우 예시:

```bash
192.168.0.1 - - [19/Feb/2025:10:15:42 +0000] "GET /index.html HTTP/1.1" 200 1234 "-" "Mozilla/5.0"
```

🚨 만약 아무 로그도 찍히지 않는다면 요청 자체가 Nginx까지 도달하지 않았을 가능성이 높다. 이 경우 방화벽 설정(`ufw`), 포트(`netstat`), 서버 바인딩 IP를 확인해 봐야 한다.

```bash
sudo netstat -tulnp | grep nginx  # Nginx가 80번 포트를 리스닝하고 있는지 확인
```

---

### Error Log 확인 (경로 문제, 파일 누락 등)

```bash
sudo tail -f error.log
```

- 경로 문제 예시:

```bash
2025/02/19 10:20:00 [error] 12345#12345: *1 open() "/usr/share/nginx/html/index.htm" failed (2: No such file or directory)
```

위와 같은 오류가 발생하면 설정 파일(`/etc/nginx/conf.d/default.conf`)에서 `root` 경로를 확인하고, 실제 파일이 존재하는지 체크해야 한다.

```bash
ls -l /usr/share/nginx/html/index.html

```

파일이 존재하지 않으면 파일을 추가하거나 `root` 경로를 올바르게 수정하면 된다.

### 4️. 추가 디버깅: SELinux 및 방화벽 설정 확인

Nginx가 실행 중인데도 페이지가 정상적으로 로딩되지 않는다면, SELinux나 방화벽 문제일 수 있다.

### SELinux 확인

```bash
getenforce
```

`Enforcing` 모드일 경우, SELinux가 파일 접근을 차단했을 가능성이 있다.
이를 일시적으로 비활성화하려면 다음 명령어를 실행하자.

```bash
sudo setenforce 0
```

영구적으로 비활성화하려면 `/etc/selinux/config` 파일을 수정하면 된다.

```bash
sudo vi /etc/selinux/config
# SELINUX=enforcing → SELINUX=disabled 변경 후 저장
```

### 5. 방화벽 설정 확인

Nginx의 기본 포트(80, 443)가 방화벽에서 허용되어 있는지 확인한다.

```bash
sudo ufw status
```

포트가 차단되어 있다면 다음 명령어로 허용하자.

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw reload

```


### 디버깅 요약 - 문제 해결 프로세스 정리

1️⃣ **Nginx 서비스 상태 확인**

```bash
sudo systemctl status nginx
```

2️⃣ **Nginx 설정 문법 검사**

```bash
sudo nginx -t
```

3️⃣ **Access Log 확인**

```bash
sudo tail -f /var/log/nginx/access.log
```

4️⃣ **Error Log 확인**

```bash
sudo tail -f /var/log/nginx/error.log
```

5️⃣ **SELinux 및 방화벽 확인**

```bash
getenforce
sudo ufw status
```

위 단계를 차례대로 확인하면 대부분의 Nginx 문제를 해결해보자~

