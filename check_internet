#!/bin/bash

PROGNAME=$(basename "$0")
CACHE_FILE=~/.cache/run_ping/state.txt

usage() {
  echo 'Run a connectivity test using ping, which tests after a sleep interval and displays error after error count'
  echo "Usage: $PROGNAME [options]"
  cat <<_EOF_

  -a        Address to ping(default=8.8.8.8)
  -s        Interval between pings(default=3s)
  -e        Errors before action(deafult=3)
  -p       Notification priority(default=critical)
  -t       Timeout in seconds at which to expire the notification, only when priority other than critical(default=5s)
_EOF_
}

quit() {
  printf '%s\n' "$1" >&2
  exit 1
}

function valid_ip() {
  local ip=$1
  local stat=1
  local IFS='.'

  if [[ $ip =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
    ip=($ip)
    [[ 10#${ip[0]} -le 255 && 10#${ip[1]} -le 255 && 10#${ip[2]} -le 255 && 10#${ip[3]} -le 255 ]]
    stat=$?
  fi
  return $stat
}

function valid_number() {
  if [[ $1 =~ ^[0-9]+$ ]]; then
    return 0
  else
    return 1
  fi
}

function displaytime {
  local T=$1
  local D=$((T / 60 / 60 / 24))
  local H=$((T / 60 / 60 % 24))
  local M=$((T / 60 % 60))
  local S=$((T % 60))
  (($D == 1)) && printf '%d day ' $D
  (($D > 1)) && printf '%d days ' $D
  printf "%02d:%02d:%02d" $H $M $S
}

while :; do
  case "$1" in
  -a)
    if valid_ip $2; then
      ping_address=$2
      shift
    else
      quit 'Error: "-a" requires a valid ip address'
    fi
    ;;
  -s)
    if valid_number $2; then
      sleep_interval=$2
      shift
    else
      quit 'Error: "-s" requires a number'
    fi
    ;;
  -e)
    if valid_number $2; then
      errors=$2
      shift
    else
      quit 'Error: "-e" requires a number'
    fi
    ;;
  -np)
    notification_priority=$2
    shift
    ;;
  -et)
    if valid_number $2; then
      timeout=$2
      shift
    else
      quit 'Error: "-et" requires a number'
    fi
    ;;
  --)
    shift
    break
    ;;
  -?*)
    usage
    quit "Error: Unknown option "$1""
    ;;
  *)
    break
    ;;
  esac
  shift
done

#Set script defaults
[ "$ping_address" ] || ping_address=8.8.8.8
[ "$sleep_interval" ] || sleep_interval=3
[ "$errors" ] || errors=3
[ "$notification_priority" ] || notification_priority='critical'

if [ ! "$timeout" ]; then
  timeout=5000
else
  timeout=$(($timeout * 1000))
fi

conn_error=0
declare -i count=0

#Only send notifications if notify-send present
if command -v notify-send &>/dev/null; then
  notify_send_present=0
else
  notify_send_present=1
fi

echo "Pinging $ping_address in $sleep_interval second intervals with a $errors error threshold"

while [ true ]; do
  if ! ping -c1 $ping_address >&/dev/null; then
    if [[ $count -lt $errors ]]; then
      ((count += 1))
    fi
    if [[ $conn_error -eq 0 && $count -eq $errors ]]; then
      start=$SECONDS
      echo -e "\033[31mNo Internet : $(date)\033[0m"
      if [[ $notify_send_present -eq 0 ]]; then
        notify-send -u $notification_priority -t $timeout 'ERROR!' 'No Internet' --icon=dialog-information
      fi
      conn_error=1
    fi
  else
    if [[ $count -gt 0 ]]; then
      ((count -= 1))
    fi
    if [[ $conn_error -eq 1 && $count -eq 0 ]]; then
      duration=$((SECONDS - start))
      echo -e "\033[32mConnected   : $(date) \033[34m(Outage\u2248$(displaytime $duration))\033[0m"
      if [[ $notify_send_present -eq 0 ]]; then
        notify-send -u $notification_priority -t $timeout 'SUCCESS' 'Internet Connection Established' --icon=dialog-information
      fi
      conn_error=0
    fi

  fi

  sleep $sleep_interval

done
