# timezone
Ansible-Playbook to change TimeZone Linux server (Ubuntu)

There is an ansible playbook that you can run it on your environment to change Timezone of servers to what ever you want.
First of all the playbook gather_facts and "Display the current timestamp in YYYY-MM-DD" of each server in hosts.ini.
After that Display hostname, current time and timezone, and using  
```yaml
when: ansible_date_time.tz != "+0330" 
```
is a conditional statement that checks whether the timezone of the target system is not equal to +0330. The ansible_date_time.tz variable contains the timezone of the target system in the format of +HHMM or -HHMM. In this case, the condition checks if the timezone is not +0330, which is the timezone for Iran.

In line which task name is:
```yaml
- name: Set timezone to Asia/Tehran
```
