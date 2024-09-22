# Zabbix Speedtest template

## Dependencies

- [bc](https://www.gnu.org/software/bc/manual/html_mono/bc.html)
- [jq](https://stedolan.github.io/jq/)
- [speedtest-cli](https://www.speedtest.net/apps/cli)

## ⚠️ Warning

You need to install [Ookla's version of speedtest-cli](https://www.speedtest.net/apps/cli) and *NOT* the unofficial python tool.

## Installation (Generic x86_64)

- Install [speedtest-cli](https://www.speedtest.net/apps/cli)
- Create `/etc/zabbix/bin`: `mkdir -p /etc/zabbix/bin`
- Copy `zbx-speedtest.sh` to `/etc/zabbix/bin`
- Make it executable: `chmod +x /etc/zabbix/bin/zbx-speedtest.sh`
- Install the systemd service and timer: `cp systemd/{zabbix-speedtest.service,zabbix-speedtest.timer} /etc/systemd/system`
- Start and enable the timer: `systemctl enable --now zabbix-speedtest.timer`
- Import the zabbix-agent config: `cp zabbix_agentd.d/speedtest.conf /etc/zabbix/zabbix_agentd.d`
- Restart zabbix-agent: `systemctl restart zabbix-agent`
- Import `template_speedtest.xml` on your Zabbix server

## Installation (Debian/Ubuntu)

- Install [speedtest-cli](https://www.speedtest.net/apps/cli)
- Create `/etc/zabbix/bin`: `mkdir -p /etc/zabbix/bin`
- Copy `zbx-speedtest-debian.sh` to `/etc/zabbix/bin/zbx-speedtest.sh`
- Make it executable: `chmod +x /etc/zabbix/bin/zbx-speedtest.sh`
- Install the systemd service and timer: `cp systemd/{zabbix-speedtest-debian.service,zabbix-speedtest.timer} /etc/systemd/system; mv /etc/systemd/system/zabbix-speedtest{-debian,}.service`
- Start and enable the timer: `systemctl enable --now zabbix-speedtest.timer`
- Import the zabbix-agent config: `cp zabbix_agentd.d/speedtest.conf /etc/zabbix/zabbix_agentd.conf.d`
- Restart zabbix-agent: `systemctl restart zabbix-agent`
- Import `template_speedtest.xml` on your Zabbix server

## Installation (OpenWRT)

- Install [speedtest-cli](https://www.speedtest.net/apps/cli) by placing the binary in your `$PATH`
- Copy `zbx-speedtest.sh` to `/etc/zabbix_agentd.conf.d/bin`
- Make it executable: `chmod +x /etc/zabbix_agentd.conf.d/bin/zbx-speedtest.sh`
- Import the zabbix-agent config: `cp zabbix_agentd.d/speedtest.openwrt.conf /etc/zabbix_agentd.conf.d`
- Restart zabbix-agent: `/etc/init.d/zabbix-agentd restart`
- Install the cron job: `crontab -e` -> Add the content of `systemd/speedtest.crontab`
- Import `template_speedtest.xml` on your Zabbix server

## Installation (Docker)

###  Speedtest in a container

Check out pschmitt/speedtest:cron on [Docker Hub](https://hub.docker.com/repository/docker/pschmitt/speedtest/general)

### Zabbix-agent 

- You must mount `zbx-speedtest.sh` inside your zabbix-agent container
- It also needs to have access to speedtest data volume

Below is an example `docker-compose.yaml`.

**NOTE:** pschmitt/zabbix-agent2 contains jq which is required by `zbx-speedtest.sh`.

```yaml
---
version: "3.7"
services:
  speedtest:
    image: pschmitt/speedtest:cron
    volumes:
      - "./data/speedtest:/data"
    environment:
      - INTERVAL=300

  zabbix-agent:
    image: pschmitt/zabbix-agent2:latest
    restart: unless-stopped
    hostname: ${HOSTNAME}
    privileged: true
    network_mode: host
    pid: host
    volumes:
      - "./config/bin:/zabbix/bin:ro"
      - "./config/zabbix_agentd.d:/etc/zabbix/zabbix_agentd.d:ro"
      - "./data/speedtest:/data/speedtest:ro"
    environment:
      - ZBX_HOSTNAMEITEM=system.hostname
      - ZBX_SERVER_HOST=zabbix.example.com
```
