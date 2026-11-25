# Change TIMEZONE
Ansible playbook that sets the timezone of Linux servers (Ubuntu) to whatever you want.

For each server in `hosts.ini`, the playbook:
1. Gathers facts and displays the current date, hostname, time and timezone.
2. Upgrades `tzdata` to the latest version from the apt repository. This runs **before** the timezone is set, because the zone must exist in tzdata and outdated tzdata gives wrong offsets. For example, old tzdata still applies daylight saving time to `Asia/Tehran`, which Iran abolished in 2022. This matters especially on Ubuntu 18.04.
3. Sets the timezone to `timezone_name` (default `Asia/Tehran`). The task is idempotent: hosts already in that timezone are not changed.

Set your timezone in the `vars:` section of `time_zone.yml`, or pass it on the command line. Valid names are listed by `timedatectl list-timezones`.

Create your host inventory and run the playbook:
```bash
ansible-playbook -i hosts.ini time_zone.yml                                   # timezone + NTP
ansible-playbook -i hosts.ini time_zone.yml --tags timezone                   # timezone only
ansible-playbook -i hosts.ini time_zone.yml --tags timezone -e timezone_name=Europe/Berlin
```

Requires the `community.general` collection: `ansible-galaxy collection install community.general`

# Sync time with your own NTP servers
The same playbook also makes sure every host's clock is synchronized by `systemd-timesyncd`, using the NTP servers listed in [ntp-pools.conf](ntp-pools.conf):

1. Runs `timedatectl status` on all hosts and reads `System clock synchronized`.
2. Only on hosts where it is `no`: appends the content of `ntp-pools.conf` to `/etc/systemd/timesyncd.conf` as an Ansible-managed block. Re-runs update that block in place instead of appending it again, and a backup of the original file is kept next to it.
3. Enables and restarts `systemd-timesyncd`, then polls `timedatectl status` until the clock is synchronized (up to 60 s by default).
4. Prints the final `timedatectl status` of every host and fails any host that is still not synchronized, showing its last `systemd-timesyncd` log lines.

Put your servers in `ntp-pools.conf` (it must have a `[Time]` header and a non-empty `NTP=` line):
```ini
[Time]
NTP=0.asia.pool.ntp.org 1.asia.pool.ntp.org 2.asia.pool.ntp.org 3.asia.pool.ntp.org
FallbackNTP=time.google.com time.cloudflare.com
```
> `FallbackNTP=` is used **only when no `NTP=` server is configured at all**. It is *not* a backup for unreachable `NTP=` servers, so put every server you want tried in `NTP=`.

```bash
ansible-playbook -i hosts.ini time_zone.yml                              # timezone + NTP
ansible-playbook -i hosts.ini time_zone.yml --tags ntp                   # NTP only
ansible-playbook -i hosts.ini time_zone.yml --tags ntp --check --diff    # dry run, changes nothing
ansible-playbook -i hosts.ini time_zone.yml --tags ntp -e ntp_force=true # also push config to synced hosts
ansible-playbook -i hosts.ini time_zone.yml --tags ntp -e ntp_pools_file=/path/to/other.conf
```

| Variable | Default | Description |
|---|---|---|
| `ntp_pools_file` | `<playbook dir>/ntp-pools.conf` | NTP servers file on the Ansible control node |
| `ntp_force` | `false` | Apply the NTP config even to hosts that are already synchronized |
| `ntp_sync_retries` / `ntp_sync_delay` | `12` / `5` | How long to wait for sync after the restart (retries × seconds) |

Requirements and notes:
- Hosts must use `systemd-timesyncd` (default on Ubuntu). Hosts managed by chrony or ntpd fail with a clear message and are left unchanged, because installing `systemd-timesyncd` would remove those services.
- Hosts need DNS resolution for the servers and outbound **UDP/123**. A firewall blocking UDP/123 is the most common reason a host stays unsynchronized.
- Active `NTP=` lines that already exist in `timesyncd.conf` or in `/etc/systemd/timesyncd.conf.d/*.conf` are **added to** these servers, not replaced.
