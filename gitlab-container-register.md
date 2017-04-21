## 安装要求:

* Docker Distribution &gt;= 2.4
* Docker GitLab &gt;= 8.8.5-1

## 配置解释

| Parameter | Description |
| :--- | :--- |
| `GITLAB_REGISTRY_ENABLED` | `true`or`false`. 是否打开Gitlab_registry的功能. |
| `GITLAB_REGISTRY_HOST` | Registry的host名称 |
| `GITLAB_REGISTRY_PORT` | Registry监听的端口 |
| `GITLAB_REGISTRY_API_URL` | Registry内部的API地址 |
| `GITLAB_REGISTRY_KEY_PATH` | Registry密钥key的路径，对应Registry的`rootdirectory` |
| `GITLAB_REGISTRY_CERT_PATH` | Registry密钥cert的路径 |
| `GITLAB_REGISTRY_ISSUER` | Registry的issuer验证,要与Registry相同|
| `SSL_REGISTRY_KEY_PATH` | 通过nginx代理https的key密钥 |
| `SSL_REGISTRY_CERT_PATH` | 通过nginx代理https的Cert密钥|

## 安装

### 1. 证书生成
请参考 issuer #14

### 2. 配置gitlab的Resistry
将生成的证书 映射 到容器目录中，把以下 配置 加入到 gitlab/docker-compose.yaml文件中
```
 gitlab:
   volumes:
   - /vagrant/www.rdc.carllhw.com:/certs
   environment:
   - GITLAB_REGISTRY_ENABLED=true
   - GITLAB_REGISTRY_HOST=registry.rdc.carllhw.com
   - GITLAB_REGISTRY_PORT=443
   - GITLAB_REGISTRY_API_URL=https://registry.rdc.carllhw.com
   - GITLAB_REGISTRY_CERT_PATH=/certs/fullchain.pem
   - GITLAB_REGISTRY_KEY_PATH=/certs/privkey.pem
```

### 3. 配置docker Resistry
在你的docker-compose.yaml文件中增加如下配置：
```
registry:
    restart: always
    image: registry.saas.hand-china.com/tools/registry:2.4.1
    ports:
    - "5000:5000"
    volumes:
    - /root/sm/registry-data:/var/lib/registry
    - /vagrant/www.rdc.carllhw.com:/certs
    environment:
    - REGISTRY_LOG_LEVEL=info
    - REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/var/lib/registry
    - REGISTRY_AUTH_TOKEN_REALM=https://gitlab.rdc.carllhw.com/jwt/auth
    - REGISTRY_AUTH_TOKEN_SERVICE=container_registry
    - REGISTRY_AUTH_TOKEN_ISSUER=gitlab-issuer
    - REGISTRY_AUTH_TOKEN_ROOTCERTBUNDLE=/certs/fullchain.pem
    - REGISTRY_STORAGE_DELETE_ENABLED=true
    - REGISTRY_HTTP_TLS_CERTIFICATE=/certs/fullchain.pem
    - REGISTRY_HTTP_TLS_KEY=/certs/privkey.pem
    - REGISTRY_HTTP_SECRET=long-and-random-alphanumeric-string
```

### 4. 配置nginx转发
如下是gitlab 和 registry的ngnix的conf配置：

* `gitlab.conf`
  
```
server {
        server_name gitlab.rdc.carllhw.com;
        listen 443 ssl;

        ssl on;
        ssl_session_timeout 5m;
        ssl_protocols SSLv2 SSLv3 TLSv1;
        #make sure you already have this certificate pair!
        ssl_certificate /cert/fullchain.pem;
        ssl_certificate_key /cert/privkey.pem;
        ssl_session_cache shared:SSL:10m;
        location /{
            client_max_body_size 0;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
	    proxy_set_header X-Forwarded-Proto https;
            proxy_redirect off;
            proxy_connect_timeout      2400;
            proxy_send_timeout         2400;
            proxy_read_timeout         2400;
	   #proxy_pass http://172.20.0.66:8000/gitlab;
	    proxy_pass http://192.168.59.103:10080;
	}
    }

```
* `registry.conf`
```
server {
        server_name registry.rdc.carllhw.com;
        listen 443 ssl;

        ssl on;
        ssl_session_timeout 5m;
        ssl_protocols SSLv2 SSLv3 TLSv1;
        #make sure you already have this certificate pair!
        ssl_certificate /cert/fullchain.pem;
        ssl_certificate_key /cert/privkey.pem;
        ssl_session_cache shared:SSL:10m;
        location /{
            client_max_body_size 0;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
	    proxy_set_header X-Forwarded-Proto https;
            proxy_redirect off;
            proxy_connect_timeout      2400;
            proxy_send_timeout         2400;
            proxy_read_timeout         2400;
	   #proxy_pass http://172.20.0.66:8000/gitlab;
	    proxy_pass https://192.168.59.103:5000;
	}
    }

```