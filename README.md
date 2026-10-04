# BT05 - Thiết lập thư mục Web an toàn tại phân vùng hệ thống (/opt/)

## 1. Tạo cấu trúc thư mục trong /opt/my-app/

```bash
devops@0310263186:~$ sudo mkdir -p /opt/my-app/html /opt/my-app/logs
devops@0310263186:~$ echo "<h1>My App</h1>" | sudo tee /opt/my-app/html/index.html
<h1>My App</h1>
```

## 2. Phân quyền sở hữu và quyền truy cập

```bash
devops@0310263186:~$ sudo chown -R devops:www-data /opt/my-app
devops@0310263186:~$ sudo find /opt/my-app -type d -exec chmod 755 {} \;
devops@0310263186:~$ sudo find /opt/my-app -type f -exec chmod 644 {} \;
devops@0310263186:~$ sudo chmod 775 /opt/my-app/logs
```

## 3. Cấu hình Server Block Nginx

```bash
devops@0310263186:~$ sudo nano /etc/nginx/sites-available/my-app.conf
devops@0310263186:~$ cat /etc/nginx/sites-available/my-app.conf
server {
    listen 80;
    listen [::]:80;
    server_name _;

    root /opt/my-app/html;
    index index.html;

    access_log /opt/my-app/logs/access.log;
    error_log  /opt/my-app/logs/error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
devops@0310263186:~$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
devops@0310263186:~$ sudo systemctl reload nginx
```

## 4. Kiểm tra phân quyền

```bash
devops@0310263186:~$ ls -la /opt/my-app/
total 16
drwxr-xr-x 4 devops www-data 4096 Oct  4 15:30 .
drwxr-xr-x 3 root   root     4096 Oct  4 15:28 ..
drwxr-xr-x 2 devops www-data 4096 Oct  4 15:30 html
drwxrwxr-x 2 devops www-data 4096 Oct  4 15:30 logs

devops@0310263186:~$ ls -la /opt/my-app/html/
total 12
drwxr-xr-x 2 devops www-data 4096 Oct  4 15:30 .
drwxr-xr-x 4 devops www-data 4096 Oct  4 15:30 ..
-rw-r--r-- 1 devops www-data   18 Oct  4 15:30 index.html
```

Ghi file bằng user devops không dùng sudo:

```bash
devops@0310263186:~$ echo "Update Test" >> /opt/my-app/html/index.html
devops@0310263186:~$ cat /opt/my-app/html/index.html
<h1>My App</h1>
Update Test
```

## 5. Kiểm tra log

```bash
thinh@local:~$ curl -I http://160.30.160.103/
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Sun, 04 Oct 2026 15:33:10 GMT
Content-Type: text/html
Content-Length: 28

devops@0310263186:~$ tail -f /opt/my-app/logs/access.log
115.76.49.152 - - [04/Oct/2026:15:33:10 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/8.5.0"
```

```bash
devops@0310263186:~$ ls -la /opt/my-app/logs/
total 16
drwxrwxr-x 2 devops www-data 4096 Oct  4 15:33 .
drwxr-xr-x 4 devops www-data 4096 Oct  4 15:30 ..
-rw-r--r-- 1 www-data www-data  112 Oct  4 15:33 access.log
-rw-r--r-- 1 www-data www-data    0 Oct  4 15:33 error.log
```
