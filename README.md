# Check-Internet

A Simple Bash Script to check internet availability and keep track of outages with notifications

```
Run a connectivity test using ping
Usage: check_internet.sh [options]

  -a        Address to ping (default=8.8.8.8)
  -s        Interval between pings (default=3s)
  -e        Errors before action (deafult=3)
  -p        Notification priority (default=critical)
  -t        Timeout in seconds at which to expire the notification, only when priority other than critical (default=5s)
```
