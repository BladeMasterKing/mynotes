# ubuntu 安装 openvpn

```bash
sudo apt install openvpn
```

```bash
whereis openvpn
```

设置服务端提供的 ovpn 文件放到 `/etc/openvpn` 目录下

在 `/etc/openvpn` 目录下创建 `passwd` 文件，第一行为账号，第二行为密码

```bash
sudo vim /usr/lib/systemd/system/openvpn.service

[Unit]
Description=OpenVPN service
After=network.target

[Service]
Type=simple
RemainAfterExit=yes
ExecStart= /usr/sbin/openvpn --config /etc/openvpn/VPNConfigBJ.ovpn --auth-user-pass /etc/openvpn/passwd
WorkingDirectory=/etc/openvpn

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload

sudo systemctl start openvpn.service

sudo systemctl status openvpn.service
```