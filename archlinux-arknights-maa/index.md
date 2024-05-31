# 在 Archlinux 上通过 ReDroid 容器运行明日方舟并使用 MAA 自动刷图


本文记录了我在我的 Archlinux 上通过 ReDroid 容器运行明日方舟并使用 MAA 进行自动刷图。

## 所需包的安装
```bash
sudo pacman -S linux-zen docker scrcpy
```
- `linux-zen`: 启用了 binderfs 模块供 ReDroid 使用
- `docker`: 用于运行 ReDroid 容器
- `scrcpy`: 显示并控制安卓设备, 它依赖的 android-tools 中的 adb 也是要用到的

## 相关配置
### linux-zen
由于我使用的 bootloader 是 systemd-boot, 所以只需要在 `/boot/loader/entries` 中新建一个 `arch-zen.conf` 并写入以下内容:
```
title	Arch Linux-zen
linux	/vmlinuz-linux-zen
initrd  /intel-ucode.img
initrd	/initramfs-linux-zen.img
options	root=UUID=... rw ibt=off nowatchdog
```

其中 `UUID` 的值是根分区的 UUID.

重启后不断按 `↓` 键选择 `Arch Linux-zen` 后启动即可, 想要默认使用此项可以将 `/boot/loader/loader.conf` 中 `default` 的值改为 `arch-zen.conf`.

### docker
将当前用户加入 `docker` 组:
```bash
sudo usermod -aG docker $USER
```

这样在使用 `docker` 命令是就不需要加上 `sudo`.

使用 `sudo systemctl start docker` 启动 docker 服务, 或者使用 `sudo systemctl enable --now docker` 启动 docker 并设置为开机自启。

## ReDroid 容器的启动
使用以下命令启动 ReDroid 容器:
```bash
docker run -d --privileged \
  -v ~/redroid11:/data \
  -p 5555:5555 \
  --name redroid11 \
  redroid/redroid:11.0.0-latest \
  androidboot.redroid_width=1280 \
  androidboot.redroid_height=720 \
  androidboot.redroid_gpu_mode=host \
  androidboot.use_memfd=1
```

第一次启动需要下载 ReDroid 镜像, 需要等待一定时间。

第一次成功启动之后再次启动可以使用 `docker start redroid11`.

使用 `docker stop redroid11` 关闭该容器。

## adb 和 scrcpy 的连接和明日方舟的安装
使用 `adb connect localhost:5555` 连接至 ReDroid 容器。

使用 `scrcpy -s localhost:5555 --no-audio` 获取容器的屏幕并进行操控。

在[明日方舟的官网](https://ak.hypergryph.com/#index)下载明日方舟的安卓版安装包之后, 使用 `adb -s localhost:5555 install ~/Downloads/arknights-...` 安装明日方舟。

安装完成后在 scrcpy 的窗口中下滑就可以看见, 启动并下载资源后登陆。

## MAA 的配置与使用
在 [MAA 的 github releases](https://github.com/MaaAssistantArknights/MaaAssistantArknights/releases) 中下载最新版的 MAA (例如 2024-05-31 最新版是 v5.3.1, 所以下载  MAA-v5.3.1-linux-x86_64.tar.gz 这个文件).

`tar -xf` 解压后进入 `Python` 目录, 其中的 `smaple.py` 是要运行的 python 文件, 将其中的 `if asst.connect('adb.exe', '127.0.0.1:5555'):` 改成 `if asst.connect('adb', '127.0.0.1:5555'):`.

在我的使用中发现直接 `python smaple.py` 会卡住, 通过调试发现是卡在 `asst/updater.py` 中 `Updater` 的 `__init__` 中获取 MAA 当前版本的地方。

将 `asst/updater.py` 中的 `self.cur_version = q.get()` 改成 `self.cur_version = "v5.3.1"` 即可, 具体的值用下载到的版本, 并将以下内容注释掉:
```py
q = queues.Queue(1, ctx=multiprocessing)
p = Process(target=self._get_cur_version, args=(path, q,))
p.start()
p.join()
```

使用 `python smaple.py` 即可正常启动, 其中的具体配置参见 [MAA 集成文档](https://maa.plus/docs/%E5%8D%8F%E8%AE%AE%E6%96%87%E6%A1%A3/%E9%9B%86%E6%88%90%E6%96%87%E6%A1%A3.html)。

之后便可以开始愉快的挂机了。

## 参考
[MAA 用户手册](https://maa.plus/docs/)
[ReDroid+MAA：在Linux下游玩明日方舟](https://lbqaq.top/p/redroid-maa/)
[使用 redroid 玩耍 Arknights](https://blog.sww.moe/post/redroid-arknights/)
