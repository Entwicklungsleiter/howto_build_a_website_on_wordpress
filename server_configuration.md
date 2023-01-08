# Server configuration

Note: This document is part of the documentation "[Howto build a website on wordpress](./README.md)".
You might want to start from the very beginning of it.

## Server configuration overview

- apt update && apt upgrade
- create another user (non root), allow sudo, import ssh key, remove ssh access and key for root
- move ssh to another port
- install docker & docker compose
- install helpful software: terminal editor (nano, vim), tmux, htop
- mount volume
- create folder structure
- add swap
- configure docker-data
- restart
- check open ports
- cron jobs
