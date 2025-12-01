# [Hysteria 2](https://github.com/apernet/hysteria) 一键脚本 安装指南

hysteria2 纯净版一键安装

```
wget -N --no-check-certificate https://raw.githubusercontent.com/Kulapichia/hysteria2-install/main/hy2/hysteria.sh && bash hysteria.sh
```
v2rayN设置中socks5监听端口: 5678


如果安装失败，请首先用VPS的后台系统重装系统后再行安装，若依然无法安装，请DD系统后安装即可。

```
bash <(wget --no-check-certificate -qO- 'https://raw.githubusercontent.com/MoeClub/Note/master/InstallNET.sh') -d 11 -v 64 -p 123456 -port 22
```
-P 为密码
-port 为服务器端口
请自行设定


详细安装地址，可参考博客内容：https://blog.yab1n.link/

## Systemd 服务启动失败排错

在将 Hysteria2 设置为 systemd 服务后，可能会遇到启动失败的问题。最常见的两大原因是：**端口冲突** 和 **文件缺失/权限问题**。请根据你遇到的具体问题，参考以下解决方案。

首先，检查服务状态和日志是定位问题的关键：
```bash
# 查看服务当前状态
systemctl status hysteria-server

# 实时查看服务日志
journalctl -u hysteria-server -f
```

### 1. 端口冲突 (Port Conflict)

#### 问题描述
Hysteria2 启动时需要监听一个 UDP 端口，如果这个端口已经被其他程序占用，Hysteria2 将无法启动。日志中通常会包含 `address already in use` 或类似的错误信息。

#### 解决方案
1.  **查找占用端口的程序**：
    假设 Hysteria2 配置的端口是 `36712`，使用以下命令查找哪个进程占用了它：
    ```bash
    ss -lunp | grep 36712
    ```
    该命令会显示占用此 UDP 端口的进程名和 PID。

2.  **解决冲突**：
    *   **方案 A：停止冲突程序**。如果占用端口的程序不是必需的，可以将其停止或卸载。
    *   **方案 B：更换 Hysteria2 端口**。使用本脚本的“修改配置”功能，或手动编辑 Hysteria2 的配置文件 `/etc/hysteria/config.yaml`，修改 `listen` 字段为你选择的一个未被占用的新端口。
        ```yaml
        # 例如，将端口从 36712 修改为 45821
        listen: :45821
        ```
    修改配置后，重启 Hysteria2 服务使其生效：
    ```bash
    systemctl restart hysteria-server
    ```

### 2. 文件缺失或权限问题 (File Missing or Permission Issues)

#### 问题描述
Systemd 服务 `hysteria-server.service` 默认以一个低权限用户 `hysteria` 运行。如果它无法读取配置文件、证书文件，或者 Hysteria2 二进制文件本身丢失，服务将启动失败。日志中可能会出现 `permission denied` 或 `no such file or directory` 的错误。

#### 解决方案
1.  **检查文件是否存在**：
    确认配置文件和证书/密钥文件是否存在于您在 `/etc/hysteria/config.yaml` 中指定的路径。
    *   **配置文件**: `/etc/hysteria/config.yaml`
    *   **证书/密钥**: 检查 `config.yaml` 中的 `tls.cert` 和 `tls.key` 字段指向的路径是否正确，并且文件确实存在。
    *   **二进制文件**: 确保 `/usr/local/bin/hysteria` 文件存在且可执行。

2.  **检查并修复文件权限**：
    `hysteria` 用户必须拥有对配置文件和证书文件的读取权限。
    *   **检查权限**: 使用 `ls -l <文件路径>` 查看文件所有者和权限。例如：
        ```bash
        ls -l /etc/hysteria/config.yaml
        ls -l /root/cert.crt 
        ```
    *   **修复权限**: 
        *   **最佳实践**: 建议将所有相关文件（配置、证书、密钥）都存放在 `/etc/hysteria` 目录下，并设置正确的所有权和权限：
            ```bash
            # 将所有权赋予 hysteria 用户和组
            chown -R hysteria:hysteria /etc/hysteria
          
            # 为配置文件和公钥证书设置合理的读取权限
            chmod 644 /etc/hysteria/config.yaml
            chmod 644 /etc/hysteria/your_cert.crt
          
            # 为私钥设置更严格的权限，仅限所有者读取
            chmod 600 /etc/hysteria/your_private.key
            ```
        *   **临时方案 (如证书在/root下)**: 如果你使用 Acme 脚本将证书生成在 `/root` 目录下，`hysteria` 用户默认无法访问。脚本通过 `chmod a+x /root` 允许了其他用户进入 `/root` 目录，但这并非最安全的方式。如果服务因权限问题启动失败，请确保证书文件本身也对 `hysteria` 用户可读。
  
    修改权限后，重新加载 Systemd 并重启服务：
    ```bash
    systemctl daemon-reload
    systemctl restart hysteria-server
    ```

---
感谢：
Ptechgithub佬 https://github.com/Ptechgithub/hysteria-install
