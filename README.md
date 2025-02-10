# Wireless Networking at Farmhill Learning Community

This repository contains the scripts, config files and documentation for setting up wireless network at [Farmhill Learning Community](https://farmhill-learning.org/).

The wireless network at Farmhill is based on OpenWrt, an open-source operating system for wireless routers based on Linux.

## Overview

Many of the wifi routers have enough compute power to directly run various software on them.

The current setup requires:

* a wifi router capable of running openwrt and have a USB port for external storage
* a USB external hard disk or flash drive

and the following things will be configured in the wifi router.

* connect to an existing wifi network to provide internet connectvity
* configured as wifi access point
* run webserver to serve static content
* run syncthing to sync content from various devices in the local network
* run periodic tasks to build the static websites hosted on the router

## The Setup

See [docs](docs/index.md).
