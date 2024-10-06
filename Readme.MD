# Information
weewx-wh23xx

Copyright 2016 Matthew Wall

Fixes: UserBogD, EdwinGH

License: GPLv3

This is a weewx driver for Fine Offset WH23xx hardware.  These stations are commonly available as Tycon TP2700, Froggit WH4000, Renkforce WH2315 (Conrad) weather stations.

# Installation

1. install weewx, select 'Simulator' driver (as placeholder)

See http://weewx.com/docs/usersguide.htm#installing

Note that I installed on Raspberry Pi model 3+ (Buster) as follows:
* Add the weewx apt repository key
`wget -qO - http://weewx.com/keys.html | sudo apt-key add -`
* Add the weewx repository
`sudo wget -qO - https://weewx.com/apt/weewx-python3.list | sudo tee /etc/apt/sources.list.d/weewx.list`
* Install weewx
`sudo apt install weewx`
* I also had to default to Python3 and install pymysql as I store the data on a remote MySQL server  
`sudo apt-get install python3-pymysql`

2. download the driver

`wget -O weewx-wh23xx.zip https://github.com/EdwinGH/weewx-wh23xx/archive/master.zip`

3. install the driver

`sudo wee_extension --install weewx-wh23xx.zip`

4. give access to non-root user to usb interface

An added line to the weewx.rules file changes the access mode to the right USB Vendor / Product combination
```
SUBSYSTEM=="usb", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="8468", MODE="0666"
```

This line was added to the weewx.rules file on this project; copy the whole file to the udev rules location
`sudo cp weewx.rules /etc/udev/rules.d`

Make sure to either reboot or run
`sudo udevadm control --reload-rules`
and replug the device to make it work.

5. configure the driver (now select wh23xx, it should appear in the list of drivers)

`sudo wee_config --reconfigure`

I had to make some updates to the config file myself, for example remove the port. My wh23xx section of the weewx.conf looks as follows:
```
[WH23xx]
    # This section is for Fine Offset WH23xx stations    
    # The model name such as Tycon, or TP2700
    model = Renkforce WH2315
    # The driver to use:
    driver = user.wh23xx
```
# Startup

1. systemd service file

I replaced the LSB init.d file with a proper systemd service file in /etc/systemd/system:
```
# systemd configuration for weewx
# created by EZ to replace /etc/init.d/weewx (removed this file)
[Unit]
Description=weewx weather system
#Requires=mysqld.service
After=time-sync.target syslog.target network.target

RequiresMountsFor=/home
# The following two lines are not in the current distribution version, but should be uncommented and used if you
# have changed the [Service] section to automatically restart the weewx service if it crashes. As noted below,
# this can be particularly useful if weewx has an IP connection to the weather station it is monitoring, since
# transient WiFi network problems are quite common, and may cause weewx to crash.
# StartLimitIntervalSec=100
# StartLimitBurst=5

[Service]
# Locations taken from /etc/defaults/weewx
ExecStart=/usr/bin/weewxd --daemon --pidfile=/run/weewx.pid /etc/weewx/weewx.conf
ExecReload=/bin/kill -HUP $MAINPID
Type=forking
PIDFile=/run/weewx.pid

# The following two lines are not in the current distribution version, but may be uncommented and used if you
# want the weewx service to automatically restart if it crashes. This can be particularly useful if weewx
# has an IP connection to the weather station it is monitoring, since transient network problems are quite
# common, and may cause the daemon to crash.
#Restart=on-failure
#RestartSec=10
Restart=always
RestartSec=60
#StartLimitInterval=400
#StartLimitBurst=3

[Install]
WantedBy=multi-user.target
```
Then added by:
`sudo systemctl enable --now weewx.service`
`sudo systemctl daemon-reload`

2. Activating weewx

`sudo systemctl start weewx`
