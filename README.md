# Git on Windows Utils

## 一、ssh-pageant

在 Windows 上使用私钥进行 `git` 免密访问的时候，有时候会出现私钥权限不对，无法使用的情况，所以可以使用 `ssh-pageant` 把 `putty` 的 `pageant` 中加载的密钥共享给 `Git for Windows` 中使用。

在 `~/.bashrc` 中加入

```bash
# ssh-pageant
eval $(/usr/bin/ssh-pageant -r -a "/tmp/.ssh-pageant-$USER")
```

## 二、pcat

`pcat` 用来使 `ssh` 通过代理来连接。

> 如果使用的是 `ssh` 协议（类似这种形式 `git@xxx.com:xxx/xxx.git`）来克隆仓库，则需要一个工具让 `ssh` 走代理。

### 安装

在 [release](https://github.com/ddxl/git-on-windows-utils/releases/latest) 中下载对应的版本，如果是 `windows` 系统，则下载 [pacat-amd64.exe](https://github.com/ddxl/git-on-windows-utils/releases/latest/download/pcat-amd64.exe)，并重命名为 `pcat.exe`，然后放入 `C:\Windows\system32` 中。

### 使用

`pcat` 支持使用两种代理，一种是 `http`，另一种是 `socks5`，通过 `--proxy` 参数来指定具体使用的代理协议，如：

-   `http` 代理，则使用 `--proxy http://xxx:xxx`

-   `socks5` 代理，则使用 `--proxy socks5://xxx:xxxx`

### 配置

在 `~/.ssh/config` 中加入

```ssh_config
Host xxx.com
    ProxyCommand pcat --proxy socks5://xxxx:1080 %h %p
```

### 另一种模式的代理

`pcat` 还支持一种特殊的代理，这种代理可以使用免费的 `SSH over Cloudflare CDN` 账户来使用，在 `Google` 上搜索 `ssh websocket`，可以搜索到一些免费的代理节点网站，可以免费新建账户，这种账户可以用来做端口转发。

-   配置文件

    ```
    Host *
        UserKnownHostsFile /dev/null
        StrictHostKeyChecking no
        TCPKeepAlive yes
        ServerAliveInterval 20
        # Ciphers chacha20-poly1305@openssh.com
        Port 80
        ProxyCommand pcat --proxy ws://%h:%p
    ```

-   动态转发

    ```shell
    # 开启一个本地的socks5服务，-l 后面为用户名，-NTD 后面为开启的socks5端口，为1080
    ssh -F /path/of/config/file xxxx -l xxxx -NTD 0.0.0.0:1080
    ```

除了使用免费的账户之外，还可以使用 `pcat` 搭建类似的代理来使用，因为免费的账户稳定性很差，所以可以用 `pcat` 来建立自己的服务。

-   配置 `/etc/ssh/sshd_config`

    ```
    Match User pcat-*
        PasswordAuthentication yes
        PermitEmptyPasswords no
    ```

-   新增一个只用来做转发无法登录 shell 的账户

    ```bash
    # 新增用户
    adduser pcat-xxx --shell=/bin/false --no-create-home
    # 设置密码
    password pcat-xxx
    ```

-   转发服务

    > 需要下载对应系统的 pcat，并安装到对应的目录

    ```shell
    # 端口最好是cloudflare的http回源端口，这里使用的是2082，如果回源为非cloudflare支持的，需要重写回源规则。
    pcat --listen ws+tcp://0.0.0.0:2082 127.0.0.1 22
    ```

-   `cloudflare`

    使用一个域名解析到服务器，并开启小云朵后即可使用。
