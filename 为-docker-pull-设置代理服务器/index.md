# 为 Docker Pull 设置代理服务器


在使用 `docker pull` 下载镜像时, 有时会遇到下载很慢的情况, 除了设置其他的镜像源外, 还可以使用代理的方式。但是在使用过程中发现 `docker pull` 时并不经过系统代理或其他常见设置下指定的代理服务器, 在参考了 [Configure the daemon with systemd](https://docs.docker.com/config/daemon/systemd/) 后得到以下两种解决办法。

## docker.service
使用 `sudo vim /usr/lib/systemd/system/docker.service` 或者 `sudo systemctl edit docker.service` 编辑 `docker.service` 在 `[Service]` 下加入以下内容:
```
Environment="HTTP_PROXY=http://127.0.0.1:12345"
Environment="HTTPS_PROXY=http://127.0.0.1:12345"
```

之后使用 `sudo systemctl restart docker` 重启 docker 即可。

注意在使用 `systemctl edit` 时, 原有的内容前会被加上井号, 不用在意, 新加的内容不需要加井号。

## daemon.json
在 `/etc/docker/daemon.json` 中写入以下内容:
```
{
  "proxies": {
    "http-proxy": "http://127.0.0.1:12345",
    "https-proxy": "http://127.0.0.1:12345"
  }
}
```

之后使用 `sudo systemctl restart docker` 重启 docker 即可。
