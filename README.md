# NGINX as a Load Balancer with Dockerized Backend Apps

This project demonstrates how NGINX can be used as a **reverse proxy and load balancer** to distribute traffic across multiple backend services running in Docker containers. It includes SSL setup, HTTPS redirection, and load balancing using the **least connections** algorithm.

---

## 📌 Project Highlights

- **HTTPS with self-signed certificates**
- **Reverse proxy and routing using NGINX**
- **Load balancing across 3 app instances**
- **Docker Compose setup for backend services**
- **Basic CI-ready structure**

---

## Docker Compose Setup

Here is the Docker Compose file used to run three backend instances of the same app on different ports. Each app responds with its name to help visualize load balancing.

```yaml
version: '3'
services:
  app1:
    build: .
    environment:
      - APP_NAME=App1
    ports:
      - "3001:3000"

  app2:
    build: .
    environment:
      - APP_NAME=App2
    ports:
      - "3002:3000"

  app3:
    build: .
    environment:
      - APP_NAME=App3
    ports:
      - "3003:3000"
```

## NGINX Configuration
```nginx configuration
upstream nodejs_cluster {
    least_conn;
    server 192.168.56.11:3001;
    server 192.168.56.11:3002;
    server 192.168.56.11:3003;
}

server {
    listen 443 ssl;
    server_name 192.168.56.11;

    ssl_certificate /path/to/cert.crt;
    ssl_certificate_key /path/to/key.key;

    location / {
        proxy_pass http://nodejs_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

server {
    listen 8080;
    location / {
        return 301 https://$host$request_uri;
    }
}
```

> ## start and stop nginx server
> ```shell
> sytemctl start nginx
> systemctl enable nginx
> systemctl status nginx
> systemctl restart nginx
> systemctl stop nginx
> ```


> ## print logs
> ```shell
>  tail -f /usr/local/var/log/nginx/access.log
>```


> ## create folder for nginx certificates
> ```shell
> mkdir ~/nginx-certs
> cd ~/nginx-certs
>```
> ## create self-signed certificate
> ```shell
> openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt
>```

