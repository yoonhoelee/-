# Azure VM을 사용한 iOS 앱 배포 서버 구축 가이드
회사에서 작업을 진행하면서 클라이언트 팀에게 전달받은 ipa 파일을 배포해야할 일이 발생하였다.
현재 다니고 있는 회사가 Azure을 사용하고 있기 때문에 Azure과 Let's Encrypt에서 무료로 받을 수 있는 인증서를 사용해 설정하는 방법에 대해서 정리를 하려고 합니다.

## 목차
1. Azure VM 생성 및 초기 설정
2. 웹 서버(Nginx) 설정
3. SSL 인증서 설정
4. iOS 앱 배포 파일 구성
5. 문제 해결 및 주의사항

## 1. Azure VM 생성 및 초기 설정

### 1.1 Azure CLI를 통한 리소스 생성
```bash
# Azure CLI 로그인
az login

# 리소스 그룹 생성 (한국 중부 리전)
az group create --name {리소스그룹명} --location koreacentral

# VM 생성 (가장 저렴한 B1s 사양 사용)
az vm create \
    --resource-group {리소스그룹명} \
    --name {VM명} \
    --image Ubuntu2204 \
    --size Standard_B1s \
    --admin-username azureuser \
    --generate-ssh-keys \
    --public-ip-sku Standard \
    --os-disk-size-gb 32 \
    --location koreacentral
```

### 1.2 필요한 포트 개방
```bash
az network nsg rule create \
    --resource-group {리소스그룹명} \
    --nsg-name {NSG명} \
    --name Allow-Web \
    --protocol tcp \
    --priority 200 \
    --destination-port-ranges 80 443 \
    --access Allow
```

## 2. 웹 서버(Nginx) 설정

### 2.1 Nginx 설치
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install nginx certbot python3-certbot-nginx -y
```

### 2.2 디렉토리 구조 설정
```bash
sudo mkdir -p /var/www/ios-apps/{앱이름}
sudo chown -R azureuser:azureuser /var/www/ios-apps
```

### 2.3 Nginx 설정 파일 생성
```bash
sudo vi /etc/nginx/sites-available/ios-apps
```

```nginx
server {
    listen 443 ssl;
    server_name {도메인 또는 IP주소};

    ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    
    client_max_body_size 500M;
    
    root /var/www/ios-apps;
    index index.html;

    location ~ \.ipa$ {
        add_header Content-Type "application/octet-stream";
        add_header Content-Disposition "attachment";
    }

    location ~ \.plist$ {
        add_header Content-Type "text/xml";
    }

    location / {
        try_files $uri $uri/ =404;
    }
}

server {
    listen 80;
    server_name {도메인 또는 IP주소};
    return 301 https://$server_name$request_uri;
}
```

### 2.4 Nginx 설정 활성화
```bash
sudo ln -s /etc/nginx/sites-available/ios-apps /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

죄송합니다. Let's Encrypt 설정 부분을 추가하겠습니다. SSL 인증서 섹션을 다음과 같이 수정하겠습니다:

## 3. SSL 인증서 설정

### 3.1 Let's Encrypt 설치 및 설정
```bash
# Certbot 및 Nginx 플러그인 설치
sudo apt update
sudo apt install -y certbot python3-certbot-nginx

# nip.io 도메인을 위한 Let's Encrypt 인증서 발급
sudo certbot --nginx -d {IP주소}.nip.io
```

### 3.2 Let's Encrypt 인증서 자동 갱신 설정
```bash
# 인증서 자동 갱신 테스트
sudo certbot renew --dry-run

# 크론잡 확인 (자동으로 설치됨)
sudo ls -l /etc/cron.d/certbot
```

### 3.3 (대체 방법) 자체 서명 인증서 생성
```bash
# 개발/테스트 환경에서만 사용 권장
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/private/nginx-selfsigned.key \
-out /etc/ssl/certs/nginx-selfsigned.crt \
-subj "/C=KR/ST=Seoul/L=Seoul/O=Test/CN={도메인 또는 IP주소}" \
-addext "subjectAltName=DNS:{도메인 또는 IP주소}"
```

### 3.4 인증서 설정 후 Nginx 업데이트
Let's Encrypt를 사용하는 경우 Certbot이 자동으로 Nginx 설정을 업데이트합니다. 수동으로 확인/수정이 필요한 경우:

```bash
sudo vi /etc/nginx/sites-available/ios-apps
```

```nginx
server {
    listen 443 ssl;
    server_name {IP주소}.nip.io;

    # Let's Encrypt 인증서 경로 (자동으로 설정됨)
    ssl_certificate /etc/letsencrypt/live/{IP주소}.nip.io/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/{IP주소}.nip.io/privkey.pem;

    # SSL 설정 최적화
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # HSTS 설정 (선택사항)
    add_header Strict-Transport-Security "max-age=31536000" always;

    root /var/www/ios-apps;
    index index.html;
    ...
}
```

### 3.5 주의사항
- Let's Encrypt 인증서는 90일마다 갱신이 필요하지만, 설치된 certbot이 자동으로 처리합니다
- nip.io 도메인을 사용하면 별도의 도메인 구매 없이 SSL 인증서를 발급받을 수 있습니다
- 실제 운영 환경에서는 자체 도메인을 사용하는 것을 권장합니다
- 자체 서명 인증서는 iOS 앱 배포 시 신뢰성 문제가 있으므로 테스트 용도로만 사용해야 합니다


## 4. iOS 앱 배포 파일 구성

### 4.1 manifest.plist 파일 생성
```bash
sudo vi /var/www/ios-apps/{앱이름}/manifest.plist
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>items</key>
    <array>
        <dict>
            <key>assets</key>
            <array>
                <dict>
                    <key>kind</key>
                    <string>software-package</string>
                    <key>url</key>
                    <string>https://{도메인 또는 IP주소}/{앱이름}/{앱파일명}.ipa</string>
                </dict>
            </array>
            <key>metadata</key>
            <dict>
                <key>bundle-identifier</key>
                <string>{번들 ID}</string>
                <key>bundle-version</key>
                <string>{버전}</string>
                <key>kind</key>
                <string>software</string>
                <key>title</key>
                <string>{앱 이름}</string>
                <key>subtitle</key>
                <string>{부제목}</string>
            </dict>
        </dict>
    </array>
</dict>
</plist>
```

### 4.2 설치 페이지 생성
```bash
sudo vi /var/www/ios-apps/{앱이름}/index.html
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>앱 설치</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body style="text-align: center; padding: 20px;">
    <h1>앱 설치</h1>
    <a href="itms-services://?action=download-manifest&url=https://{도메인 또는 IP주소}/{앱이름}/manifest.plist" 
       style="display: inline-block; padding: 10px 20px; background-color: #007AFF; color: white; text-decoration: none; border-radius: 5px;">
        앱 설치하기
    </a>
</body>
</html>
```

### 4.3 IPA 파일 업로드 및 권한 설정
```bash
# 로컬에서 VM으로 IPA 파일 전송
scp {앱파일명}.ipa azureuser@{VM IP주소}:/home/azureuser/

# VM에서 파일 이동 및 권한 설정
sudo mv /home/azureuser/{앱파일명}.ipa /var/www/ios-apps/{앱이름}/
sudo chown www-data:www-data /var/www/ios-apps/{앱이름}/{앱파일명}.ipa
sudo chmod 644 /var/www/ios-apps/{앱이름}/{앱파일명}.ipa
```

## 5. 문제 해결 및 주의사항

### 5.1 SSL 인증서 관련
- iOS는 신뢰할 수 있는 SSL 인증서가 필요합니다
- 테스트를 위해 nip.io 도메인을 사용하여 Let's Encrypt 인증서를 발급받을 수 있습니다
- 실제 배포 환경에서는 proper 도메인과 SSL 인증서를 사용하는 것을 권장합니다

### 5.2 파일 권한 문제
- 모든 파일에 대해 적절한 권한이 설정되어 있어야 합니다
- www-data 사용자가 파일에 접근할 수 있어야 합니다

### 5.3 iOS 개발자 프로필
- 개발용 IPA의 경우 디바이스 UDID가 프로비저닝 프로파일에 포함되어 있어야 합니다
- 설치 후 Settings > General > Device Management에서 개발자 신뢰 설정이 필요합니다

## 참고 사항
- VM 생성 시 최소 사양(B1s)으로도 충분합니다
- nginx의 client_max_body_size는 IPA 파일 크기에 따라 적절히 조정이 필요합니다
- 보안을 위해 필요한 포트(80, 443)만 개방하는 것을 권장합니다
- 실제 운영 환경에서는 도메인과 유효한 SSL 인증서 사용을 강력히 권장합니다
