---
layout: post
title: 3 ways to create start up daemon in linux!
---
## Creating a script as a service
Three init script framework available right now. I will describe them one by one. Init is the first process when kernel gets loaded. Then leave the starting up process for process like systemd or systemvinit
### 1. System V Init
System V Init comes around since 1983. Then is became de defacto choice of all the POSIX like systsem. The key to SystemVInit is inittab. Inittabs organise the process into different run-levels:
```
- 0 for halt
- 1 (S) for single user mode
- 3 for normal booting (multi-user mode)
- 5 for X-Server
- 6 for reboot
```
A process in inittab is presented like this:
```
id:runlevels:action:process
```
System V init uses **/etc/init.d** scripts to control the start, stop, and restart of daemons. An example is provided below:
```
#!/bin/bash
#
# Script to run Shadowsocks in daemon mode at boot time.
#====================================================================
# Run level information:
# Do update-rc.d [this filename]
# Then service [this filename] enable
#====================================================================

#====================================================================
# Paths and variables and system checks.

# Source function library
. /etc/rc.d/init.d/functions

# Check that networking is up.
#
[ NETWORKING ="yes" ] || exit 0

# Daemon

# Path to the lock file.
#
LOCK_FILE=/var/lock/subsys/shadowsocks

# Path to the pid file.
#
PID=/var/run/NAME/pid


#====================================================================

#====================================================================
# Run controls:

RETVAL=0

# Start shadowsocks as daemon.
#
start() {
# Check lock so we don't run the same process twice
if [ -f LOCK_FILE ]; then
echo "NAME is already running!"
exit 0
else
echo -n $"Starting NAME: "
#daemon --check DAEMON --user USER "DAEMON -f PID -c CONF > /dev/null"
/usr/local/bin/ssserver -c /etc/shadowsocks/config.json
RETVAL=$?
[ RETVAL -eq 0 ] && success
echo
[ RETVAL -eq 0 ] && touch LOCK_FILE
return RETVAL
}


# Stop shadowsocks.
#
stop() {
echo -n $"Shutting down NAME: "
killproc -p PID
RETVAL=$?
[ RETVAL -eq 0 ]
rm -f LOCK_FILE
rm -f PID
echo
return RETVAL
}

# See how we were called.
case "" in
start) 
  start 
  ;; 
stop) 
  stop 
  ;;
restart)
  stop 
  start 
  ;;
condrestart)
if [ -f LOCK_FILE ]; then
  stop
  start
  RETVAL=$?
  fi
  ;;
status)
  status DAEMON
  RETVAL=$?
  ;;
*)
  echo $"Usage:  {start|stop|restart|condrestart|status}"
  RETVAL=1
esac

exit RETVAL#
```
### 2. Upstart
Upstart is dsigend with backward compatibilty. It runs daemon without any modifications to the startup scripts.
To tell upstart what to do, you need to define jobs. A jobs is triggerd by events, common events are starting, started, stopping and stopped.
Jobs configuration files are written in **/etc/init/*.conf**
>[https://www.digitalocean.com/community/tutorials/the-upstart-event-system-what-it-is-and-how-to-use-it](https://www.digitalocean.com/community/tutorials/the-upstart-event-system-what-it-is-and-how-to-use-it)

**例子**
```
description "Test node.js server"
author      "Your Name"

start on filesystem or runlevel [2345]
stop on shutdown

script

    export HOME="/srv"
    echo $$ > /var/run/nodetest.pid // Find an available pid
    exec /usr/bin/nodejs /srv/nodetest.js

end script

pre-start script
    echo "[`date`] Node Test Starting" >> /var/log/nodetest.log
end script

pre-stop script
    rm /var/run/nodetest.pid
    echo "[`date`] Node Test Stopping" >> /var/log/nodetest.log
end script
```
### 3. Systemd
Systemd is a suite of basic building blocks for a Linux system. It provides a system and service manager that runs as PID 1 and starts the rest of the system.
We use systemctl to control systemd, the following commands might come in handy.
```
systemctl status \ list all status of the services

systemctl \ list running services

systemctl --failed \ list failed services
```
The systemctl unit files are stored in two folders:
1. /usr/lib/systemd/system for units provided by installed packages
2. /etc/systemd/system for user installed units

A simple example for server to start shadowsocks.
```
[Unit]
Description=Shadowsocks Server
After=network.target
//After=multi-user.target

[Service]
ExecStart=/usr/local/bin/ssserver -c /etc/shadowsocks/ss-config.json
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

)
