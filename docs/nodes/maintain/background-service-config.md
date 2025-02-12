---
tags: [Nodes]
description: This page demonstrates how to set up a `avalanchego.service` file to enable a manually deployed validator node to run in the background of a server instead of in the terminal directly.
sidebar_label: Run as a Background Service
pagination_label: Run an Avalanche Node as a Background Service
sidebar_position: 6
---
# Run an Avalanche Node as a Background Service

## Overview

This page demonstrates how to set up a `avalanchego.service` file to
enable a manually deployed validator node to run in the background of
a server instead of in the terminal directly.

## Prerequisites

- AvalancheGo installed

## Steps

### Fuji Testnet Config

Run this command in your terminal to create the `avalanchego.service` file

```shell
sudo nano /etc/systemd/system/avalanchego.service
```

Paste the following configuration into the `avalanchego.service` file

***Remember to modify the values of:***

- ***user=***
- ***group=***
- ***WorkingDirectory=***
- ***ExecStart=***

***For those that you have configured on your Server***

```shell
[Unit]
Description=Avalanche Node service
After=network.target

[Service]
User='YourUserHere'
Group='YourUserHere'
Restart=always
PrivateTmp=true
TimeoutStopSec=60s
TimeoutStartSec=10s
StartLimitInterval=120s
StartLimitBurst=5
WorkingDirectory=/Your/Path/To/avalanchego
ExecStart=/Your/Path/To/avalanchego/./avalanchego \  
   --network-id=fuji \
   --api-metrics-enabled=true 

[Install]
WantedBy=multi-user.target
```

Press **Ctrl + X** then **Y** then **Enter** to save and exit.

Now, run:

```shell
sudo systemctl daemon-reload
```

### 2) Mainnet Config

Run this command in your terminal to create the `avalanchego.service` file

```shell
sudo nano /etc/systemd/system/avalanchego.service
```

Paste the following configuration into the `avalanchego.service` file

```shell
[Unit]
Description=Avalanche Node service
After=network.target

[Service]
User='YourUserHere'
Group='YourUserHere'
Restart=always
PrivateTmp=true
TimeoutStopSec=60s
TimeoutStartSec=10s
StartLimitInterval=120s
StartLimitBurst=5
WorkingDirectory=/Your/Path/To/avalanchego
ExecStart=/Your/Path/To/avalanchego/./avalanchego \
   --api-metrics-enabled=true

[Install]
WantedBy=multi-user.target
```

Press **Ctrl + X** then **Y** then **Enter** to save and exit.

Now, run:

```shell
sudo systemctl daemon-reload
```

## Start the Node

This command makes your node start automatically in case of a reboot, run it:

```shell
sudo systemctl enable avalanchego
```

To start the node, run:

```shell
sudo systemctl start avalanchego
sudo systemctl status avalanchego
```

Output:

```Lua
socopower@avalanche-node-01:~$ sudo systemctl status avalanchego
● avalanchego.service - Avalanche Node service
     Loaded: loaded (/etc/systemd/system/avalanchego.service; enabled; vendor p>
     Active: active (running) since Tue 2023-08-29 23:14:45 UTC; 5h 46min ago
   Main PID: 2226 (avalanchego)
      Tasks: 27 (limit: 38489)
     Memory: 8.7G
        CPU: 5h 50min 31.165s
     CGroup: /system.slice/avalanchego.service
             └─2226 /usr/local/bin/avalanchego/./avalanchego --network-id=fuji

Aug 30 03:02:50 avalanche-node-01 avalanchego[2226]: INFO [08-30|03:02:50.685] >
Aug 30 03:02:51 avalanche-node-01 avalanchego[2226]: INFO [08-30|03:02:51.185] >
Aug 30 03:03:09 avalanche-node-01 avalanchego[2226]: [08-30|03:03:09.380] INFO >
Aug 30 03:03:23 avalanche-node-01 avalanchego[2226]: [08-30|03:03:23.983] INFO >
Aug 30 03:05:15 avalanche-node-01 avalanchego[2226]: [08-30|03:05:15.192] INFO >
Aug 30 03:05:15 avalanche-node-01 avalanchego[2226]: [08-30|03:05:15.237] INFO >
Aug 30 03:05:15 avalanche-node-01 avalanchego[2226]: [08-30|03:05:15.238] INFO >
Aug 30 03:05:19 avalanche-node-01 avalanchego[2226]: [08-30|03:05:19.809] INFO >
Aug 30 03:05:19 avalanche-node-01 avalanchego[2226]: [08-30|03:05:19.809] INFO >
Aug 30 05:00:47 avalanche-node-01 avalanchego[2226]: [08-30|05:00:47.001] INFO
```

To see the synchronization process, you can run the following command:

```shell
sudo journalctl -fu avalanchego
```
