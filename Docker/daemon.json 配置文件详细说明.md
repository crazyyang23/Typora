# Docker `daemon.json` 配置文件详细说明

`daemon.json` 是 Docker 守护进程的配置文件，用于配置 Docker 引擎的各种行为。该文件通常位于以下路径：

- **Linux 系统**: `/etc/docker/daemon.json`
- **Windows 系统**: `C:\ProgramData\docker\config\daemon.json`

如果文件不存在，可以手动创建。

------

## 配置文件结构

`daemon.json` 是一个 JSON 格式的文件，包含一系列键值对，用于配置 Docker 守护进程的行为。以下是一些常见的配置选项：

| 配置项                     | 说明                                                         |
| :------------------------- | :----------------------------------------------------------- |
| `data-root`                | 指定 Docker 镜像、容器、卷等数据的存储路径。默认路径为 `/var/lib/docker`。 |
| `log-level`                | 设置 Docker 守护进程的日志级别。可选值包括 `debug`, `info`, `warn`, `error`, `fatal`。 |
| `log-driver`               | 设置容器的日志驱动。常见的日志驱动包括 `json-file`, `syslog`, `journald`, `gelf`, `fluentd`, `awslogs` 等。 |
| `log-opts`                 | 配置日志驱动的选项。例如，可以设置日志文件的最大大小和最大数量。 |
| `insecure-registries`      | 允许 Docker 使用不安全的私有镜像仓库（HTTP）。               |
| `registry-mirrors`         | 配置 Docker 镜像仓库的镜像地址，用于加速镜像拉取。           |
| `dns`                      | 设置容器的 DNS 服务器。                                      |
| `dns-opts`                 | 配置 DNS 选项。                                              |
| `dns-search`               | 设置 DNS 搜索域。                                            |
| `default-ulimits`          | 设置容器的默认资源限制（如文件描述符数量、进程数等）。       |
| `live-restore`             | 允许 Docker 守护进程在重启时保持容器的运行状态。             |
| `storage-driver`           | 设置 Docker 使用的存储驱动。常见的存储驱动包括 `overlay2`, `aufs`, `btrfs`, `zfs` 等。 |
| `exec-opts`                | 配置容器的执行选项。                                         |
| `bip`                      | 设置 Docker 网桥的 IP 地址。                                 |
| `fixed-cidr`               | 设置 Docker 网桥的子网。                                     |
| `mtu`                      | 设置 Docker 网络的 MTU（最大传输单元）。                     |
| `default-shm-size`         | 设置容器的默认共享内存大小。                                 |
| `max-concurrent-downloads` | 设置同时下载镜像的最大并发数。                               |
| `max-concurrent-uploads`   | 设置同时上传镜像的最大并发数。                               |
| `experimental`             | 启用或禁用 Docker 的实验性功能。                             |

------

## 示例配置文件

以下是一个 `daemon.json` 的示例配置：

json

复制

```
{
  "data-root": "/mnt/docker-data",
  "log-level": "info",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "insecure-registries": [
    "myregistry.local:5000"
  ],
  "registry-mirrors": [
    "https://mirror.example.com"
  ],
  "dns": ["8.8.8.8", "8.8.4.4"],
  "dns-opts": ["timeout:2"],
  "dns-search": ["example.com"],
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 65536,
      "Soft": 32768
    }
  },
  "live-restore": true,
  "storage-driver": "overlay2",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "bip": "172.17.0.1/16",
  "fixed-cidr": "172.17.0.0/24",
  "mtu": 1500,
  "default-shm-size": "64m",
  "max-concurrent-downloads": 3,
  "max-concurrent-uploads": 3,
  "experimental": false
}
```

------

## 配置说明

- **`data-root`**: 将 Docker 的数据存储路径设置为 `/mnt/docker-data`。
- **`log-level`**: 设置日志级别为 `info`。
- **`log-driver`**: 使用 `json-file` 日志驱动，并设置日志文件的最大大小为 10MB，最多保留 3 个日志文件。
- **`insecure-registries`**: 允许 Docker 使用 `myregistry.local:5000` 这个不安全的私有镜像仓库。
- **`registry-mirrors`**: 配置了一个镜像仓库的镜像地址 `https://mirror.example.com`，用于加速镜像拉取。
- **`dns`**: 设置容器的 DNS 服务器为 Google 的公共 DNS 服务器 `8.8.8.8` 和 `8.8.4.4`。
- **`dns-opts`**: 设置 DNS 查询超时为 2 秒。
- **`dns-search`**: 设置 DNS 搜索域为 `example.com`。
- **`default-ulimits`**: 设置容器的默认文件描述符限制为硬限制 65536，软限制 32768。
- **`live-restore`**: 启用 `live-restore` 功能，允许 Docker 守护进程在重启时保持容器的运行状态。
- **`storage-driver`**: 使用 `overlay2` 存储驱动。
- **`exec-opts`**: 配置容器的执行选项，使用 `systemd` 作为 cgroup 驱动。
- **`bip`**: 设置 Docker 网桥的 IP 地址为 `172.17.0.1/16`。
- **`fixed-cidr`**: 设置 Docker 网桥的子网为 `172.17.0.0/24`。
- **`mtu`**: 设置 Docker 网络的 MTU 为 1500。
- **`default-shm-size`**: 设置容器的默认共享内存大小为 64MB。
- **`max-concurrent-downloads`**: 设置同时下载镜像的最大并发数为 3。
- **`max-concurrent-uploads`**: 设置同时上传镜像的最大并发数为 3。
- **`experimental`**: 禁用 Docker 的实验性功能。

------

## 应用配置

修改 `daemon.json` 后，需要重启 Docker 守护进程以使配置生效：

bash

复制

```
sudo systemctl restart docker
```

------

## 注意事项

1. 修改 `daemon.json` 文件时，确保 JSON 格式正确，否则 Docker 守护进程可能无法启动。
2. 某些配置选项可能需要特定的 Docker 版本或操作系统支持。
3. 在生产环境中修改配置前，建议先在测试环境中验证配置的正确性。

通过合理配置 `daemon.json`，可以优化 Docker 的性能、安全性和可管理性。