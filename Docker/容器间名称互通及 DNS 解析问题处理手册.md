---
version: '3'
services:
  service1:
    image: nginx
    dns:
      - 127.0.0.11  # Docker内置DNS，用于解析同一网络内的容器名称
      - 8.8.8.8  # 外部DNS服务器，用于解析公网域名
---

容器间名称互通及 DNS 解析问题处理手册（详细版）

一、问题现象细化

在容器内部，当执行ping 目标容器名称命令时，系统返回 “Temporary failure in name resolution” 错误提示。这意味着容器无法将目标容器的名称转换为对应的 IP 地址，进而导致容器之间无法通过名称进行通信。这种情况不仅会影响ping命令的使用，还可能导致基于容器名称的服务调用、数据交互等操作失败。

二、可能原因深度分析

（一）网络配置问题

容器之间的通信依赖于网络连接，若容器不在同一网络中，它们处于相互隔离的网络环境，无法直接进行数据传输和名称解析。Docker 的网络模式有多种，如 bridge（桥接网络）、host（主机网络）、overlay（覆盖网络）等，不同网络模式下容器的通信规则和名称解析方式存在差异，若配置不当，极易导致网络不通。

（二）DNS 配置问题

容器内的/etc/resolv.conf文件是 DNS 配置的关键文件，该文件中记录了 DNS 服务器的地址等信息。如果文件中 DNS 服务器地址缺失、错误，或者存在多个冲突的 DNS 服务器地址，都会导致容器无法正常解析其他容器的名称。例如，缺少 Docker 内置的 DNS 服务器地址[127.0.0.11](http://127.0.0.11/)，容器就无法利用 Docker 自身的 DNS 解析功能解析同一网络内的容器名称。

（三）DNS 服务器可达性问题

即使 DNS 配置正确，若配置的 DNS 服务器无法访问，也会导致解析失败。可能是 DNS 服务器所在的主机宕机、网络线路故障，或者存在网络访问控制策略阻止了容器与 DNS 服务器之间的通信。例如，企业内部的 DNS 服务器因维护而停止服务，容器就无法通过该服务器进行名称解析。

（四）防火墙问题

防火墙作为网络安全的重要屏障，其规则可能会阻止容器之间的 DNS 请求（通常使用 UDP 53 端口）或 ICMP 协议（ping命令基于 ICMP 协议）通信。例如，防火墙设置了拒绝所有 UDP 53 端口的入站和出站请求，容器的 DNS 解析请求就无法到达 DNS 服务器，从而导致解析失败；若阻止了 ICMP 协议，即使名称解析成功，ping命令也无法正常执行。

（五）Docker 服务问题

Docker 服务运行异常会直接影响其内置的 DNS 功能。Docker 服务启动过程中若出现错误、关键组件损坏，或者服务进程处于不稳定状态，都可能导致内置 DNS 无法正常工作，进而影响容器之间的名称解析。

三、详细排查步骤

（一）检查容器网络配置

1. **查看容器所属网络**

执行命令docker inspect 容器名称 | grep Networks，该命令会输出容器的详细网络信息，在输出结果中找到 “Networks” 关键字对应的内容，即可明确容器当前所在的网络名称。例如，若输出结果中有"Networks": {"my-network": {...}}，则说明该容器在my-network网络中。

1. **查看所有网络**

运行docker network ls命令，会列出系统中所有的 Docker 网络，包括网络 ID、名称、驱动类型和作用域等信息。通过此命令可以了解系统中存在哪些网络，为后续判断容器是否在同一网络提供依据。

1. **确认容器是否在同一网络**

对比需要互通的各个容器所属的网络名称，若不一致，则表明它们不在同一网络。只有处于同一网络的容器，才有可能通过 Docker 内置的 DNS 功能实现名称解析和通信。

（二）验证容器 DNS 配置

1. **检查容器内 DNS 配置**

执行docker exec -it 容器名称 cat /etc/resolv.conf命令，进入容器内部并查看/etc/resolv.conf文件的内容。该文件通常包含nameserver（DNS 服务器地址）、search（搜索域）等配置项。

1. **分析 DNS 配置是否正常**

正常情况下，对于使用自定义网络的容器，/etc/resolv.conf文件中应包含nameserver [127.0.0.11](http://127.0.0.11/)（Docker 内置 DNS 服务器地址），这是 Docker 为同一网络内的容器提供名称解析的关键。若文件中没有该地址，或存在多个冲突的 DNS 服务器地址，都属于 DNS 配置异常。

（三）测试容器间连通性

1. **通过 IP 地址 ping 测试**

- **获取目标容器 IP**：执行docker inspect -f '{{.NetworkSettings.Networks.网络名称.IPAddress}}' 目标容器名称命令，其中 “网络名称” 为目标容器所在的网络名称。例如，docker inspect -f '{{.[NetworkSettings.Networks.my](http://networksettings.networks.my/)-network.IPAddress}}' container2，该命令会输出目标容器在my-network网络中的 IP 地址。

- **在源容器内 ping IP**：执行docker exec -it 源容器名称 ping 目标IP，若能够收到回复，说明容器之间通过 IP 地址可以通信，问题可能出在名称解析环节；若无法收到回复，则可能是网络驱动配置错误、防火墙规则阻止等原因导致网络不通。

- version: '3'

  services:

    service1:

  ​    image: nginx

  ​    dns:

  ​      \- 127.0.0.11  # Docker内置DNS，用于解析同一网络内的容器名称

  ​      \- 8.8.8.8  # 外部DNS服务器，用于解析公网域名

1. **手动解析容器名称**

在源容器内执行docker exec -it 源容器名称 nslookup 目标容器名称命令。若命令输出中能够显示目标容器的 IP 地址，说明 DNS 解析功能正常；若提示 “** server can't find 目标容器名称: NXDOMAIN**” 等错误信息，则表明 DNS 解析存在问题。若nslookup成功但ping失败，可能是防火墙阻止了 ICMP 协议，此时可改用nc -vz 目标容器名称 端口命令测试端口连通性，例如nc -vz container2 80，若显示 “succeeded!”，说明端口可通。

（四）检查 DNS 服务器可达性

1. **测试外部 DNS 服务器**

- 执行ping 外部DNS服务器地址命令，若能够收到回复，说明容器与外部 DNS 服务器之间的网络连接正常；若超时无回复，则可能是网络不通。

- 执行nslookup google.com 外部DNS服务器地址命令，若能成功解析出google.com的 IP 地址，说明外部 DNS 服务器工作正常；若解析失败，可能是 DNS 服务器配置错误或功能异常。

1. **处理 DNS 服务器不可达问题**

若外部 DNS 服务器无法访问，首先检查 DNS 服务器地址是否正确，然后排查网络线路、路由器配置等是否存在问题。若无法解决，可更换为可靠的公共 DNS 服务器，如8.8.8.8（Google DNS）、1.1.1.1（Cloudflare DNS）等。

（五）检查 Docker 网络驱动

执行docker network inspect 网络名称 | grep Driver命令，查看网络使用的驱动类型。对于单机环境，推荐使用bridge驱动；在跨主机环境中，通常使用overlay驱动。若驱动类型不符合实际使用场景，可能会导致网络功能异常，影响容器通信和名称解析。

（六）验证 Docker DNS 服务

执行docker exec -it $(docker ps -qf "name=docker_gwbridge") nslookup 目标容器名称命令。docker_gwbridge是 Docker 默认的网关桥接网络，该命令用于检查 Docker 内置的 DNS 服务是否能够正常解析目标容器的名称。若解析成功，说明 Docker DNS 服务工作正常；若解析失败，则可能是 Docker 服务异常导致 DNS 功能失效。

（七）检查防火墙规则

1. **临时关闭防火墙测试**

- CentOS/RHEL 系统：执行sudo systemctl stop firewalld命令临时关闭防火墙。

- Ubuntu 系统：执行sudo ufw disable命令临时禁用防火墙。

关闭防火墙后，再次测试容器间的名称解析和通信。若问题解决，说明是防火墙规则导致的；若问题依旧，则防火墙不是导致问题的原因。

1. **检查并调整防火墙规则**

若确定是防火墙问题，需要开放必要的端口和协议。对于 DNS 解析，需要开放 UDP 53 端口；对于 ICMP 协议（ping命令），需要允许 ICMP 数据包通过。具体配置方法因操作系统和防火墙软件而异，例如在 CentOS/RHEL 系统中，可使用firewall-cmd命令添加规则；在 Ubuntu 系统中，可使用ufw命令进行配置。

四、详细解决方案

（一）确保容器在同一网络

1. **创建自定义网络**

执行docker network create --driver bridge 网络名称命令，创建一个桥接类型的自定义网络。例如，docker network create --driver bridge my-network，其中--driver bridge指定网络驱动为桥接模式，这是单机环境中最常用的网络模式。

1. **将容器加入网络**

- 对于已创建的容器，执行docker network connect 网络名称 容器名称命令，将容器加入指定网络。例如，docker network connect my-network container1。

- 对于新创建的容器，在启动时通过--network参数指定网络，命令为docker run -d --name 容器名称 --network 网络名称 镜像名称。例如，docker run -d --name container1 --network my-network nginx。

（二）修复 DNS 解析问题

1. **重启 Docker 服务**

执行sudo systemctl restart docker命令重启 Docker 服务。重启服务可以解决因 Docker 进程异常导致的内置 DNS 功能失效等问题。重启完成后，检查 Docker 服务状态，确保其正常运行，执行systemctl status docker命令，若显示 “active (running)”，则说明服务启动成功。

1. **重建容器并指定网络**

- 停止容器：执行docker stop 容器1 容器2命令，停止需要重建的容器。

- 删除容器：执行docker rm 容器1 容器2命令，删除已停止的容器。

- 重新创建容器：执行docker run -d --name 容器名称 --network 网络名称 镜像名称命令，使用自定义网络重新创建容器，确保容器在同一网络中。

1. **临时手动添加 hosts 映射（备选方案）**

启动容器时，通过--add-host参数手动添加目标容器的名称与 IP 地址的映射关系，命令为docker run -d --name 源容器名称 --add-host 目标容器名称:目标IP 镜像名称。例如，docker run -d --name container1 --add-host container2:172.18.0.3 nginx，其中172.18.0.3是container2的 IP 地址。这种方法适用于临时测试或特殊场景，容器重启后映射关系需要重新配置。

（三）配置 DNS 服务器

1. **仅使用 Docker 内置 DNS（推荐）**

创建自定义网络后，确保所有需要互通的容器都加入该网络。Docker 会自动为该网络中的容器配置内置 DNS 服务器127.0.0.11，容器之间可以直接通过名称进行解析和通信，无需额外配置其他 DNS 服务器。

1. **混合配置（容器内同时使用两个 DNS）**

在docker-compose.yml文件中为服务指定多个 DNS 服务器，适用于既需要解析同一网络内容器名称，又需要解析外部公网域名的场景。示例配置如下：

TypeScript取消自动换行复制version: '3'services:  service1:    image: nginx    dns:      - 127.0.0.11  # Docker内置DNS，用于解析同一网络内的容器名称      - 8.8.8.8  # 外部DNS服务器，用于解析公网域名

配置完成后，执行docker-compose up -d命令启动服务，容器会按照配置的 DNS 服务器顺序进行名称解析。

（四）开放防火墙端口

1. **CentOS/RHEL 系统**

- 执行firewall-cmd --add-port=53/udp --permanent命令，开放 UDP 53 端口，--permanent参数表示规则永久生效。

- 执行firewall-cmd --reload命令，重新加载防火墙规则，使配置生效。

1. **Ubuntu 系统**

执行ufw allow 53/udp命令，开放 UDP 53 端口，该命令会立即生效，且默认规则为永久生效。

五、详细验证步骤

（一）测试容器间名称解析

1. 执行docker exec -it 源容器名称 bash命令，进入源容器内部。

1. 在容器内部执行ping 目标容器名称命令，若能够收到连续的回复信息，说明容器之间可以通过名称正常通信；若仍然提示 “Temporary failure in name resolution” 错误，则问题未解决，需要重新排查。

（二）测试公网域名解析

1. 进入容器内部：执行docker exec -it 容器名称 bash命令。

1. 执行ping google.com命令，若能够收到回复，说明容器可以正常解析公网域名；若解析失败，检查容器的 DNS 配置和外部 DNS 服务器的可达性。

六、补充场景及处理

（一）跨主机容器通信

在跨主机环境中，需要使用overlay网络实现容器之间的通信。首先需要初始化 Docker Swarm 集群，在主节点执行docker swarm init --advertise-addr 主节点IP命令，然后在工作节点执行主节点返回的加入命令。创建overlay网络：docker network create -d overlay 网络名称，之后在各主机上启动容器时指定该网络，容器之间即可通过名称进行通信。

（二）Docker Compose 管理的容器

使用 Docker Compose 管理的容器，默认会创建一个自定义网络，服务之间可以通过服务名称进行通信。若出现名称解析问题，检查docker-compose.yml文件中是否正确配置了网络，以及服务是否加入了该网络。可以通过docker-compose down命令停止服务，再执行docker-compose up -d命令重新启动，重建网络配置。

七、预防措施细化

1. **生产环境工具选择**：生产环境中，推荐使用 Docker Compose 或 Kubernetes 管理容器。Docker Compose 适用于单机多容器应用，能够自动创建网络并配置 DNS，简化容器间的通信配置；Kubernetes 适用于大规模容器集群，提供了完善的服务发现和 DNS 解析机制，确保容器在复杂环境中稳定通信。

1. **DNS 配置规范**：严格区分不同网络环境的 DNS 配置，避免在同一容器中混用127.0.0.11（Docker 内置 DNS）和外部 DNS 服务器。对于仅需要内部通信的容器，仅使用 Docker 内置 DNS；对于需要访问外部网络的容器，合理配置外部 DNS 服务器，并确保与内置 DNS 不冲突。

1. **DNS 性能监控**：部署 CoreDNS 或 Dnsmasq 作为本地 DNS 缓存服务器，缓存常用的域名解析结果，减少对外部 DNS 服务器的依赖，提高解析效率。同时，使用监控工具（如 Prometheus+Grafana）监控 DNS 服务器的响应时间、解析成功率等指标，及时发现并解决 DNS 性能问题。

1. **定期检查机制**：制定定期检查计划，每周至少检查一次容器的网络配置（通过docker network inspect命令）和 DNS 配置（通过docker exec命令查看/etc/resolv.conf文件），确保网络正常、DNS 配置正确。同时，检查 DNS 服务器的可达性和工作状态，预防潜在的解析问题。