## Nginx 설치 및 실행 (Ubuntu)

**Ubuntu 환경에서 최신 버전의 Nginx를 설치, 실행 및 기초 학습을 위한 공간**

일반적으로 `apt install nginx` 명령어로 설치할 수도 있지만, **최신 버전을 설치하기 위해 추가 패키지 작업을 거쳤음.**

### 1. Nginx 필수 패키지 설치

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

---

### Nginx 실행 및 상태 확인

설치가 완료 되었으면, Nginx를 실행 및 상태 확인

```bash
# Nginx 상태 확인
sudo systemctl status nginx

# Nginx 실행
sudo systemctl start nginx

```

---

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
