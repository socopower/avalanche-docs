---
tags: [Tooling, AvalancheGo APIs]
description: AvalancheGo Installer is a shell (bash) script that installs AvalancheGo on any Linux computer. This script sets up full, running node in a matter of minutes with minimal user input required. This is convenient if you want to run the node as a service on a standalone Linux installation, for example to set up a (Subnet) validator, use the node as a private RPC server and similar uses. It also makes upgrading or reinstalling the nodes easy.
pagination_label: AvalancheGo Install Script
---

# AvalancheGo Installer

AvalancheGo Installer is a shell (bash) script that installs AvalancheGo on any
Linux computer. This script sets up full, running node in a matter of minutes
with minimal user input required. This is convenient if you want to run the node
as a service on a standalone Linux installation, for example to set up a
(Subnet) validator, use the node as a private RPC server and similar uses. It
also makes upgrading or reinstalling the nodes easy.

GitHub: [https://github.com/ava-labs/avalanche-docs/blob/master/scripts/avalanchego-installer.sh](https://github.com/ava-labs/avalanche-docs/blob/master/scripts/avalanchego-installer.sh)

How-to: [Run an Avalanche Node Using the Install Script](/nodes/run/with-installer.md)

If you want to run a node in a more complex environment, like in a docker or
Kubernetes container, or as a part of an installation orchestrated using a tool
like Terraform, this installer probably won't fit your purposes. See here for how to run AvalancheGo
in a [Docker container](/tooling/cli-guides/run-with-docker.md).
