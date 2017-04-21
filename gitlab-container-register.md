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
### 1.证书生成
请参考 issuer #14

### 2.配置gitlab的Resistry
将生成的证书 映射 到容器目录中
把以下 配置 加入到 gitlab/docker-compose.yml文件中
```
 gitlab:
   volumes:
    - ./certs:/certs
   environment:
    - GITLAB_REGISTRY_ENABLED=true
    - GITLAB_REGISTRY_HOST=registry.rdc.carllhw.com
    - GITLAB_REGISTRY_PORT=443
    - GITLAB_REGISTRY_API_URL=https://registry.rdc.carllhw.com
    - GITLAB_REGISTRY_CERT_PATH=/certs/fullchain.pem
    - GITLAB_REGISTRY_KEY_PATH=/certs/privkey.pem
```