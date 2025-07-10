# Docker-fnOS

真正可用可部署在Docker中的fnOS，不再让宝塔&1panel用户面临艰难的选择。

## 💫 轻松在Docker中运行你的fnOS

飞牛 fnOS 是一款极具潜力的国产免费 NAS 系统，尤其适合新手用户和家庭用户。它安装简单、界面精美易用，而且可以高效地利用闲置硬件，搭建 NAS 私人存储来替代网盘，并安装使用各种 Docker 应用。本项目旨在提供一种在 Docker 容器中便捷部署 fnOS 的方案。
![img2](https://github.com/user-attachments/assets/663e6545-f25e-40df-afab-43f91ceda9eb)
![img10](https://github.com/user-attachments/assets/9efb53bb-7e1d-4b1d-9b58-da672bc3ecf2)
![img9](https://github.com/user-attachments/assets/7393f0fb-bc5a-41df-9a90-501a21c61e55)
![img8](https://github.com/user-attachments/assets/13bc487e-82d2-4554-871c-dddd70c57d71)
![img7](https://github.com/user-attachments/assets/ddefb8e2-145f-4247-811f-7b4fe7cdf248)
![img6](https://github.com/user-attachments/assets/7d533603-87ec-456d-bee0-661448120452)
![img5](https://github.com/user-attachments/assets/6b6bb73c-435d-476a-9200-a72aeb4306b8)
![img4](https://github.com/user-attachments/assets/37e42abc-14dd-4e3a-b309-f07690fdbb41)
![img3](https://github.com/user-attachments/assets/cc4e20a1-be31-4fee-b0b8-ddf3a2d208cf)

## 💡 项目介绍

本项目提供了一种在 Docker 环境中部署和运行飞牛 fnOS NAS 系统的方法。通过将 fnOS 运行在 Docker 容器内的 QEMU 虚拟机中，可以避免直接在物理机或虚拟机上安装的复杂性，为用户提供更灵活、可移植的部署选择。

## ✨ 特性

* **🖥️ 容器化部署:** 在 Docker 环境中轻松运行和管理 fnOS。
* **🔄 基于 QEMU:** 利用成熟的 QEMU 虚拟机技术在容器内部运行 fnOS 镜像。
* **🚀 快速启动:** 通过 docker-compose 即可快速配置和启动 fnOS 虚拟机。
* **🏠 家庭友好:** 继承 fnOS 本身易用性，适合家庭用户快速搭建 NAS。
* **📦 Docker 应用支持:** 在容器化的 fnOS 中，依然可以安装和使用其自带的 Docker 应用商店。

## 🚀 快速入门

本节介绍如何使用 `docker-compose` 快速部署 fnOS。

### 概念原理

本项目利用 https://github.com/qemus/qemu 项目提供的基础镜像，在 Docker 容器内部通过 QEMU 模拟硬件环境来启动并运行 fnOS 系统。这意味着 fnOS 作为一个完整的操作系统运行在容器内的虚拟机中。

### 部署配置

使用 `docker-compose` 是最推荐的部署方式。创建一个名为 `docker-compose.yml` 的文件，并将以下内容复制进去：

```yaml
services:
  fnos:
    image: ghcr.io/qemus/qemu:7.12 # 使用包含QEMU的镜像
    container_name: fnos # 容器名称
    environment:
      BOOT: "https://iso.liveupdate.fnnas.com/x86_64/trim/fnos-0.9.11-946.iso" # 飞牛os安装镜像地址，请替换为最新稳定版本或系统盘镜像路径
      RAM_SIZE: "2G" # qemu虚拟机设定的内存大小
      CPU_CORES: "4" # qemu虚拟机设定的CPU核心数
      DISK_SIZE: "16G" # 飞牛系统盘大小
      DISK2_SIZE: "200G" # 数据盘大小，可以添加更多 DISK_SIZE=... 参数创建更多数据盘
    devices:
      - /dev/kvm # 映射 KVM 设备文件，用于启用硬件加速（如果宿主机支持）
      - /dev/net/tun # 映射 TUN/TAP 设备文件，用于网络连接
    cap_add:
      - NET_ADMIN # 添加网络管理权限，允许容器配置网络接口
    ports:
      # 将容器内部的 fnOS 端口映射到宿主机端口
      - 8006:8006 # QEMU Web管理界面默认端口映射
      - 5666:5666 # fnOS Web管理界面默认端口映射
      #- 5005:5005 # fnOS 相关WebDAV服务端口映射（可自行选择是否开启）
      #- 5006:5006 # fnOS 相关WebDAV服务端口映射（可自行选择是否开启）
      #- 445:445 # fnOS 相关SMB服务端口映射（可自行选择是否开启）
      #- 21:21 # fnOS 相关FTP服务端口映射（可自行选择是否开启）
    volumes:
      # 将宿主机目录映射到 fnOS 虚拟机内部作为数据盘，请替换 /dir1 和 /dir2 为实际路径
      - /dir1:/storage # 映射到 fnOS 内部的第一个数据盘目录
      - /dir2:/storage2 # 映射到 fnOS 内部的第二个数据盘目录
      # 你可以将 fnOS 的系统盘镜像也通过 volume 映射出来，以便持久化系统状态
      # - ./fnos_system.qcow2:/drive/fnos_system.qcow2 # 示例：将系统盘镜像文件映射出来
    restart: unless-stopped # Docker容器退出后总是重启（除非手动停止）
    stop_grace_period: 2m # 优雅停止的等待时间
    # networks: # 如需设置飞牛系统本地固定IP等，可在此配置网络，可能需要 macvlan 等模式
    #   default:
    #     ipv4_address: 192.168.1.100 # 示例固定IP (需配合 macvlan 网络配置)
```

### 配置说明

* **`image`**: 指定使用的 Docker 镜像，`ghcr.io/qemus/qemu:7.12` 是本项目基于的 QEMU 镜像。
* **`container_name`**: 为你的 fnOS 容器指定一个容易识别的名称。
* **`environment`**: 这一部分是配置 QEMU 虚拟机的关键：
  * `BOOT`: 指定启动介质。可以是 fnOS 的 ISO 安装镜像地址（用于首次安装），也可以是已经安装好的 fnOS 系统盘镜像文件路径。首次部署通常使用 ISO 地址。
  * `RAM_SIZE`, `CPU_CORES`: 分别设置分配给 fnOS 虚拟机的内存和 CPU 核心数。请根据你的宿主机资源合理分配。
  * `DISK_SIZE`, `DISK2_SIZE`: 设置 fnOS 的系统盘和数据盘大小。`DISK_SIZE` 是主系统盘，`DISK2_SIZE` 及后续 `DISKn_SIZE` 会被识别为数据盘。**注意，首次启动如果是用 ISO 安装，系统盘和数据盘会在安装过程中格式化，请确保映射的 volumes 路径下没有重要数据或使用新的空目录。**
* **`devices`**: 映射 `/dev/kvm` 和 `/dev/net/tun` 设备文件到容器内，这对于虚拟机的硬件加速和网络功能至关重要。
* **`cap_add`**: 授予容器 `NET_ADMIN` 权限，以便虚拟机内的系统可以进行网络配置。
* **`ports`**: 将容器内部（即 fnOS 虚拟机内部）的端口映射到宿主机端口。默认将 fnOS 的 Web 管理界面端口 8006 和服务端口 5666 映射出来。
  * **重要提示：** 如果你在 fnOS 内部安装了需要通过特定端口访问的应用（例如下载工具的 Web UI），你需要修改此处的 `ports` 部分，添加相应的端口映射。这可能导致与宿主机上已占用的端口发生冲突。解决冲突的方法包括修改宿主机映射端口（`宿主机端口:容器内部端口`）、修改 fnOS 内部应用的监听端口，或使用 `macvlan` 等更高级的 Docker 网络模式为 fnOS 容器分配独立的 IP 地址。
* **`volumes`**: 将宿主机的本地目录映射到 fnOS 虚拟机内部，作为 fnOS 可用的数据盘。**请务必修改 `volumes` 部分中的 `/dir1` 和 `/dir2` 为你宿主机上实际用于存储数据的目录路径！** 你也可以考虑映射 fnOS 的系统盘镜像文件，以便持久化系统状态，避免每次重启容器都重新安装。
* **`restart`**: 设置容器的重启策略，`unless-stopped` 表示除非手动停止，否则 Docker 守护进程总是会重启容器。
* **`stop_grace_period`**: 设置 Docker 停止容器时的等待时间，给虚拟机留出正常关机的时间。

### 启动

在保存了 `docker-compose.yml` 文件的目录下，打开终端并运行以下命令来启动 fnOS 容器：

```
docker-compose up -d
```

`-d` 参数表示在后台运行容器。

### 访问 fnOS

容器启动并完成 fnOS 的安装过程后（如果 BOOT 指定的是 ISO），你可以通过宿主机的 IP 地址和映射的端口（默认为 8006）在浏览器中访问 fnOS 的 Web 管理界面进行后续的配置和使用。

例如，如果你的宿主机 IP 是 `192.168.1.10`，则在浏览器中打开 `http://192.168.1.10:8006`。

![111123vjttjhh7e780rzc0](https://github.com/user-attachments/assets/41538a08-2535-4499-9756-37e7c0ed6903)
![111124kpgmwi6nc7oei36w](https://github.com/user-attachments/assets/3c798670-86f1-4c9e-b888-9d4201cc623e)
![111124u4ssdbcywh5he0r5](https://github.com/user-attachments/assets/b814e2f2-3d76-4cc9-894d-02e0fd2e9c5a)
![111125r42xk34hc435hm23](https://github.com/user-attachments/assets/be0152d4-8e6b-4916-8946-03115a4036b2)
![111125ihmsbiz3o3ymmw5z](https://github.com/user-attachments/assets/5db6ede3-86da-479d-9803-1699fbce8956)
![111125twzoe9gjqjgbgkwo](https://github.com/user-attachments/assets/5d8737a2-45b1-40e0-b88a-dc650598ea8a)
![111126e9uredemnwm9wpup](https://github.com/user-attachments/assets/1a0afcc1-4603-4c75-9928-983e92106145)
![微信截图_20250710172753](https://github.com/user-attachments/assets/2e7b876f-bd3c-46aa-bdd3-b0b4738e79da)
![111126b00yhwtwwtjptwz5](https://github.com/user-attachments/assets/a42afbe2-cce6-4314-82c8-17e55b8f8275)
![111127vrdtl15tmwtllvtg](https://github.com/user-attachments/assets/b6a00094-ffc8-4008-994a-1d152dff5564)
