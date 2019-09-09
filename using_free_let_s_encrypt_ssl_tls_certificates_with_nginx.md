# NGINX와 Let’s Encrypt를 이용하여 무료 SSL/TLS 인증하기   
> (원글: https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/)

Editor – The blog post detailing the original procedure for using Let’s Encrypt with NGINX (from February 2016) redirects here. The instructions in that post are deprecated.

Also see our blog post from nginx.conf 2015, in which Peter Eckersley and Yan Zhu of the Electronic Frontier Foundation introduce the then‑new Let’s Encrypt certificate authority.

It’s well known that SSL/TLS encryption of your website leads to higher search rankings and better security for your users. However, there are a number of barriers that have prevented website owners from adopting SSL.

Two of the biggest barriers have been the cost and the manual processes involved in getting a certificate. But now, with Let’s Encrypt, they are no longer a concern. Let’s Encrypt makes SSL/TLS encryption freely available to everyone.

Let’s Encrypt is a free, automated, and open certificate authority (CA). Yes, that’s right: SSL/TLS certificates for free. Certificates issued by Let’s Encrypt are trusted by most browsers today, including older browsers such as Internet Explorer on Windows XP SP3. In addition, Let’s Encrypt fully automates both issuing and renewing of certificates.

In this blog post, we cover how to use the Let’s Encrypt client to generate certificates and how to automatically configure NGINX Open Source and NGINX Plus to use them.

### Let’s Encrypt의 동작
Before issuing a certificate, Let’s Encrypt validates ownership of your domain. The Let’s Encrypt client, running on your host, creates a temporary file (a token) with the required information in it. The Let’s Encrypt validation server then makes an HTTP request to retrieve the file and validates the token, which verifies that the DNS record for your domain resolves to the server running the Let’s Encrypt client.

### 전제조건
Let's Encrypt를 이용하여 시작하기 전에, 필요사항: 
- NGINX 또는 NGINX Plus의 설치.
- 인증서를 위해 등록된 도메인네임의 보유 혹은 제어. 만약 등록된 도메인네임이 없으면, GoDaddy 혹은 dnsexit 등을 이용하여 등록할 수 있다.
- 서버의 공용 IP주소와 도메인네임을 연결하는 DNS 레코드를 만든다.

이제 NGINX 혹은 NGINX Plus에 대한 Let's Encrypt 설정을 쉽게할 수 있다.

**Note**: 이 포스트에 설명된 절차는 Ubuntu 16.04 (Xenial)에서 테스트되었음.

### 1. Let’s Encrypt Client를 다운로드
첫째, Let’s Encrypt client인 certbot을 다운로드:
    
1. **certbot** 저장소 생성:
```bash
$ add-apt-repository ppa:certbot/certbot
```
2. certbot 설치:
```bash
$ apt-get update
$ apt-get install python-certbot-nginx
```
이제 Let’s Encrypt client를 사용할 준비가 되었음.

### 2. NGINX 설정 
certbot은 자동적으로 NGINX SSL/TLS 설정을 할 수 있다. 이것은 NGINX 설정에서 당신이 인증서를 요청한 도메인네임에 대한 server_name 지시자를 포함하는 서버블럭을 찾아 수정한다. 이 예에서, 도메인은 **www\.example.com**이다.

당신이 새로 NGINX를 설치하여 시작한다고 가정하면, 텍스트 에디터를 이용하여 **/etc/nginx/conf.d** 디렉토리에 ***domain-name.conf***를 생성한다 (이 예에서는, **www\.example.com.conf**이다).

Specify your domain name (and variants, if any) with the server_name directive:
```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    server_name example.com www.example.com;
}
```
Save the file, then run this command to verify the syntax of your configuration and restart NGINX:

```bash
$ nginx -t && nginx -s reload
```

### 3. Obtain the SSL/TLS Certificate
The NGINX plug‑in for certbot takes care of reconfiguring NGINX and reloading its configuration whenever necessary.

Run the following command to generate certificates with the NGINX plug‑in:

$ sudo certbot --nginx -d example.com -d www.example.com
Respond to prompts from certbot to configure your HTTPS settings, which involves entering your email address and agreeing to the Let’s Encrypt terms of service.

When certificate generation completes, NGINX reloads with the new settings. certbot generates a message indicating that certificate generation was successful and specifying the location of the certificate on your server.

Congratulations! You have successfully enabled https://example.com and https://www.example.com 

-------------------------------------------------------------------------------------
IMPORTANT NOTES: 

```bash
Congratulations! Your certificate and chain have been saved at: 
/etc/letsencrypt/live/example.com/fullchain.pem 
Your key file has been saved at: 
/etc/letsencrypt/live/example.com//privkey.pem
Your cert will expire on 2017-12-12.
```
**Note**: Let’s Encrypt certificates expire after 90 days (on 2017-12-12 in the example). For information about automatically renenwing certificates, see Automatic Renewal of Let’s Encrypt Certificates below.

If you look at ***domain‑name.conf***, you see that certbot has modified it:

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    server_name  example.com www.example.com;

    listen 443 ssl; # managed by Certbot

    # RSA certificate
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot

    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot

    # Redirect non-https traffic to https
    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    } # managed by Certbot
}
```

### 4. Automatically Renew Let’s Encrypt Certificates
Let’s Encrypt certificates expire after 90 days. We encourage you to renew your certificates automatically. Here we add a cron job to an existing **crontab** file to do this.

1. Open the **crontab** file.

```bash
$ crontab -e
```
2. Add the certbot command to run daily. In this example, we run the command every day at noon. The command checks to see if the certificate on the server will expire within the next 30 days, and renews it if so. The --quiet directive tells certbot not to generate output.

```bash
0 12 * * * /usr/bin/certbot renew --quiet
```
Save and close the file. All installed certificates will be automatically renewed and reloaded.

### Summary
We’ve installed the Let’s Encrypt agent to generate SSL/TLS certificates for a registered domain name. We’ve configured NGINX to use the certificates and set up automatic certificate renewals. With Let’s Encrypt certificates for NGINX and NGINX Plus, you can have a simple, secure website up and running within minutes.

To try out Let’s Encrypt with NGINX Plus yourself, start your free 30-day trial today or contact us to discuss your use cases.

Cover image Microservices: From Design to Deployment
The complete guide to microservices development
