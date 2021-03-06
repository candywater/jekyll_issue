---
layout: post
title: about systemd service
tags: linux systemd service
---


### Service File Location

we can find many service in linux.  

Let's take a look at Nginx.service

```
$ sudo systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2019-03-31 12:50:17 UTC; 32min ago
 Main PID: 2839 (nginx)
   CGroup: /system.slice/nginx.service
           ├─2839 nginx: master process /usr/sbin/nginx
           └─2840 nginx: worker process
```

and if we type such command like this

```
$ sudo systemctl enable nginx
```
```
Created symlink from 
/etc/systemd/system/multi-user.target.wants/nginx.service to 
/usr/lib/systemd/system/nginx.service.
```

A symbolic link will be created.

### Put Your Service File Here

here is where you should write service

```
/usr/lib/systemd/system/service-name.service
```

then use the following command

```
$ sudo systemctl enable service-name
```

### Service File Layout

here is nginx service file.

```
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

### Write Your Service File

this is just a little complex.  
here is a simple one.


```
[Unit]
Description=this is a description
After=you can copy from nginx.service

[Service]
Type=simple   
ExecStart=/usr/bin/something /home/your-script
KillMode=process
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=your-program-id-in-syslog
Environment=YOUR_VAR=xxxx
Restart=always

[Install]
WantedBy=multi-user.target
```
- Type <- at most time simple is enough, sometimes you may need a forking[1](https://www.freedesktop.org/software/systemd/man/systemd.service.html)

- Environment <- this is something you may need

- Restart <- the best thing I like, your service will never down

- multi-user.target <- this is for "enable", multi-user.target which means command line use is simple output[2](https://linoxide.com/linux-how-to/install-desktop-environments-centos-7/)

<!--

```
$ cat /run/nginx.pid
2839
```

```
/etc/systemd/system/multi-user.target.wants/mysqld.service

[Unit]
...

[Install]
...

[Service]
...
...
Restart=always
...

```
```
# Restarting/reloading
systemctl daemon-reload # Run if .service file has changed
systemctl restart
```

```
Created symlink from /etc/systemd/system/multi-user.target.wants/kjc_back.service to /usr/lib/systemd/system/kjc_back.service.
```
-->

### Conclusion

Write a Original Service File, and put it into 

```
/usr/lib/systemd/system/
```

then just start & enable it via systemctl.

[How To Configure a Linux Service to Start Automatically After a Crash or Reboot – Part 1: Practical Examples ](https://www.digitalocean.com/community/tutorials/how-to-configure-a-linux-service-to-start-automatically-after-a-crash-or-reboot-part-1-practical-examples)  
[How to Auto-start Services on Boot in Linux?](https://geekflare.com/how-to-auto-start-services-on-boot-in-linux/)  
[How to set up proper start/stop services](https://blog.frd.mn/how-to-set-up-proper-startstop-services-ubuntu-debian-mac-windows/)  
[What is the difference between “systemctl start” and “systemctl enable”?](https://askubuntu.com/questions/733469/what-is-the-difference-between-systemctl-start-and-systemctl-enable)  
[Creating Systemd Service Files](https://www.devdungeon.com/content/creating-systemd-service-files)  
[what is the CentOS equivalent of /var/log/syslog (on Ubuntu)?](https://unix.stackexchange.com/questions/88744/what-is-the-centos-equivalent-of-var-log-syslog-on-ubuntu)  
[sudo service status includes bad;](https://askubuntu.com/questions/836059/sudo-service-status-includes-bad)  
[Run node.js service with systemd](https://www.axllent.org/docs/view/nodejs-service-with-systemd/)  
[How to Install Desktop Environments on CentOS 7 ](https://linoxide.com/linux-how-to/install-desktop-environments-centos-7/)  
[Install MATE or XFCE on CentOS 7](http://jensd.be/125/linux/rhel/install-mate-or-xfce-on-centos-7)  
[How To Install Xfce GUI In CentOS 7 Linux](https://www.rootusers.com/how-to-install-xfce-gui-in-centos-7-linux/)  
[man page: systemd.service — Service unit configuration](https://www.freedesktop.org/software/systemd/man/systemd.service.html)  
[man page: systemd.unit — Unit configuration](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#)  