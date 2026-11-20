# BJTU Pi hotspot

[English](README.md) | 简体中文

用一台树莓派 4 做网关：自动登录北京交通大学（BJTU）校园网认证页面，通过局域网共享网络，并在夜间自动更新。
所有配置都由 Ansible 从你的电脑一键部署到一台全新的树莓派上。

```
校园网 --> [eth0  树莓派  eth1] --> TP-Link --> 你的设备
(认证页面)    自动登录    192.168.50.1/24    热点
```

## 电脑端准备

1. 安装 `ansible-core`。
2. 安装所需集合：`ansible-galaxy collection install community.general ansible.posix`

## 部署一台全新的树莓派

1. 用 Raspberry Pi Imager 烧录 **Raspberry Pi OS Lite 64-bit**，**不要做任何自定义设置**（不建用户，不配 Wi-Fi）。
2. `cp authorized_keys.example authorized_keys`，然后填入你的 SSH 公钥（每行一个）。该文件已被 git 忽略。
3. 把 SD 卡插回电脑，然后执行：
   ```
   ansible-playbook bootstrap.yml -e boot=/path/to/bootfs
   ```
   这会在启动分区写入 `user-data`（cloud-init）：root 的 SSH 公钥（来自本地 `authorized_keys` 文件）、开启 SSH、不创建任何用户，并在 eth1 上开启热点局域网（192.168.50.1，为客户端分配 DHCP）。
4. eth0 接校园网，eth1（USB 网卡）接 TP-Link，启动树莓派，电脑连上 TP-Link 的 Wi-Fi。树莓派地址为 `192.168.50.1`。此时还没有外网（尚未通过认证）：请先关闭电脑上的 VPN。
5. `cp .env.example .env`，然后填写 `BJTU_USERNAME` 和 `BJTU_PASSWORD`。
   `.env` 已被 git 忽略，保存校园网账号密码，会被复制到树莓派的 `/etc/bjtu-login.env`（权限 0600）。
6. `inventory/hosts.yml` 已指向 `192.168.50.1`，无需修改。
7. 第一次运行（默认 SSH 端口 22）：
   ```
   ansible-playbook site.yml -e ansible_port=22
   ```
8. 之后的运行（端口 22422）：
   ```
   ansible-playbook site.yml
   ```

需要开放其他端口？在 `group_vars/all.yml` 中设置 `firewall_extra_tcp_ports` / `firewall_extra_udp_ports`。

## 部署内容

| 角色 | 作用 |
|---|---|
| `bjtu_login` | 安装 `/usr/local/bin/bjtu-login`、`/etc/bjtu-login.env`（0600）、systemd 服务 + 定时器，以及 NetworkManager 调度脚本（eth0 连上时重新登录）。如果没有外网，还会完成第一次认证登录。 |
| `base` | 时区、基础软件包，关闭不用的服务（avahi、蓝牙、Wi-Fi 无线），硬件看门狗，日志保存在内存中以保护 SD 卡 |
| `network` | 只管理 eth1：NetworkManager 连接 `lan_connection`（`hotspot-lan`），地址 `lan_address`（192.168.50.1/24），共享模式（NAT + DHCP/DNS）。eth0 保持 NetworkManager 默认的 DHCP。 |
| `firewall` | nftables 入站规则（表 `rpi_guard`）：热点客户端只能使用 DHCP、DNS 和 SSH；eth0 上不开放任何端口（包括 SSH，因为它有公网 IPv6）；可选额外端口。另有针对 SSH 的 fail2ban |
| `autoupdate` | 夜间自动安装更新（unattended-upgrades），仅在需要时重启 |
| `ssh` | 端口 22422，仅允许密钥登录。root 的 `authorized_keys` 完全由本地 `authorized_keys` 文件管理。最后执行。 |

## 夜间计划

| 时间 | 操作 |
|---|---|
| 03:00 | `apt update` |
| 04:00 | `apt upgrade`（unattended-upgrades） |
| 05:00 | 仅在更新需要时重启 |

时间可在 `group_vars/all.yml` 中修改。

## 故障排查

- 登录日志：`journalctl -u bjtu-login`（日志保存在内存中，重启后会丢失）
- 查看定时器：`systemctl list-timers`
- 手动触发登录：`systemctl start bjtu-login`
- 运行前预览改动：`ansible-playbook site.yml --check --diff`

## 许可证

[MIT](LICENSE)。欢迎贡献：请参阅 [CONTRIBUTING.md](CONTRIBUTING.md)。
