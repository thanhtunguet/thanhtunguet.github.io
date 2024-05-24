---
title: Cài đặt OpenVPN trên Ubuntu
date: 2020-04-17 19:10:00 +0700
categories: ["Devops", Networks]
tags: ["Devops", "Networks", "OpenVPN"]
pin: false
---

## Cài đặt OpenVPN với script

```bash
wget https://raw.githubusercontent.com/Nyr/openvpn-install/master/openvpn-install.sh
chmod a+x openvpn-install.sh
sudo ./openvpn-install.sh
```

## Cronjob kiểm tra kết nối

1. Tạo file script kiểm tra kết nối

```bash
sudo nano /usr/local/bin/autovpn
```

với nội dung sau:

```bash
#!/bin/bash

function getStatus () {
	ifconfig | grep $1 && return 1
	return 0
}

getStatus tun0
if [[ $? == 0 ]];
then
	nmcli con up id vpn-connection-name
fi
```

2. Gán thuộc tính executable

```bash
sudo chmod a+x /usr/local/bin/autovpn
```

3. Cấu hình cronjob với `crontab -e`

```bash
* * * * * /usr/local/bin/autovpn >/dev/null 2>&1
```
