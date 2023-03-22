# Change TIMEZONE
Ansible-Playbook to change TimeZone Linux server (Ubuntu)
There is an ansible playbook that you can run on your environment to change the Timezone of servers to whatever you want. First of all the playbook gather_facts and "Display the current timestamp in YYYY-MM-DD" of each server in hosts.ini. After that Display hostname, current time, and timezone, and use 
```yaml
when: ansible_date_time.tz != "+0330" 
```
is a conditional statement that checks whether the timezone of the target system is not equal to +0330. The ansible_date_time.tz variable contains the timezone of the target system in the format of +HHMM or -HHMM. In this case, the condition checks if the timezone is not +0330, which is the timezone for Iran.

is a conditional statement that checks whether the timezone of the target system is not equal to +0330. The ansible_date_time.tz variable contains the timezone of the target system in the format of +HHMM or -HHMM. In this case, the condition checks if the timezone is not +0330, which is the timezone for Iran.
In a line which task name is:
```yaml
- name: Set timezone to Asia/Tehran
```
Set the timezone to your desired Geographic area/Time zone
The last task upgrade package "tzdata" to the latest version from apt repository. Upgrade tzdata must be done specially on Ubuntu-18.04
Just create your host inventory and run the playbook.

```yaml
ansible-playbook -i hosts.ini time_zone.yml
```
