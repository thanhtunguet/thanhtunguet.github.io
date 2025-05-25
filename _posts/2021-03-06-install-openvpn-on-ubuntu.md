---
title: Install OpenVPN on Ubuntu
date: 2020-04-17 19:10:00 +0700
categories: ["Devops", Networks]
tags: ["Devops", "Networks", "OpenVPN"]
pin: false
---

## Install OpenVPN with a script

```bash
wget https://raw.githubusercontent.com/Nyr/openvpn-install/master/openvpn-install.sh
chmod a+x openvpn-install.sh
sudo ./openvpn-install.sh
```

## Cronjob to check connection

1. Create a script file to check the connection

```bash
sudo nano /usr/local/bin/autovpn
```

with the following content:

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

2. Set the executable attribute

```bash
sudo chmod a+x /usr/local/bin/autovpn
```

3. Configure the cronjob with `crontab -e`

```bash
* * * * * /usr/local/bin/autovpn >/dev/null 2>&1
```
