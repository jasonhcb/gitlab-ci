## 安装要求:

* Docker Distribution &gt;= 2.4
* Docker GitLab &gt;= 8.8.5-1

## 配置解释

```
    - GITLAB_REGISTRY_ENABLED=true
    - GITLAB_REGISTRY_HOST=registry.rdc.carllhw.com
    - GITLAB_REGISTRY_PORT=443
    - GITLAB_REGISTRY_API_URL=https://registry.rdc.carllhw.com
    - GITLAB_REGISTRY_CERT_PATH=/certs/fullchain.pem
    - GITLAB_REGISTRY_KEY_PATH=/certs/privkey.pem
```
| Parameter | Description |
| :--- | :--- |
| `GITLAB_REGISTRY_ENABLED` | `true`or`false`. Enables the Registry in GitLab. By default this is`false`. |
| `GITLAB_REGISTRY_HOST` | The host URL under which the Registry will run and the users will be able to use. |
| `GITLAB_REGISTRY_PORT` | The port under which the external Registry domain will listen on. |
| `GITLAB_REGISTRY_API_URL` | The internal API URL under which the Registry is exposed to. |
| `GITLAB_REGISTRY_KEY_PATH` | The private key location that is a pair of Registry's`rootcertbundle`. Read the[token auth configuration documentation](https://docs.docker.com/registry/configuration/#token). |
| `GITLAB_REGISTRY_PATH` | This should be the same directory like specified in Registry's`rootdirectory`. Read the[storage configuration documentation](https://docs.docker.com/registry/configuration/#storage). This path needs to be readable by the GitLab user, the web-server user and the Registry user_if you use filesystem as storage configuration_. Read more in[\#container-registry-storage-path](https://github.com/sameersbn/docker-gitlab/blob/master/docs/container_registry.md#container-registry-storage-path). |
| `GITLAB_REGISTRY_ISSUER` | This should be the same value as configured in Registry's`issuer`. Otherwise the authentication will not work. For more info read the[token auth configuration documentation](https://docs.docker.com/registry/configuration/#token). |
| `SSL_REGISTRY_KEY_PATH` | The private key of the`SSL_REGISTRY_CERT_PATH`. This will be later used in nginx to proxy your registry via https. |
| `SSL_REGISTRY_CERT_PATH` | The certificate for the private key of`SSL_REGISTRY_KEY_PATH`. This will be later used in nginx to proxy your registry via https. |



