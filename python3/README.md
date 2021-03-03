# Service Discovery Protocol (SDP)

Note: python-bluez3 is basically using BlueZ 4 and I have BlueZ 5. The code in rfcomm_server_spd.py is using some depricated BlueZ 4 APIs, so I currently have to force the bluetoothd to run in compatibility mode as described below.

## 1. Disable the Magic System Request Key
This can be done in a script or by editing the file /etc/sysctl.conf file. Take your pick. On my setup, I'm using the /etc/sysctl.conf method so that I don't have to worry about running a script prior to using the bluetooth device.

### 1.1 Script
Add this line to a script if you want the script to disable the magic system request key. If you use this method then you will need to ensure that the script is run before using the bluetooth device.
`sudo bash -c "echo 0 > /proc/sys/kernel/sysrq"`

### 1.2 /etc/sysctl.conf
Edit the __kernel.sysrq=438__ line in /etc/sysctl.conf to be __kernel.sysrq=0__ and make sure the line is uncommented.
`kernel.sysrq=0`

## 2. Add the Serial Port (SP) service
Add the SP service to the local sdpd of the host (e.g. Ubuntu machine) that will run the rfcomm_server_sdp.py script.
`sudo sdptool add SP`

## 3. Make sure <user> is in the bluetooth group
Run this command to see if your user is in the bluetooth group.
`cat /etc/group | grep bluetooth`

If it isn't, then add your user to the bluetooth group.
`sudo usermod -G bluetooth -a <user>`

## 4. Change group of the /var/run/sdp file
This file gets created by the bluetooth service running on the host machine everytime the bluetooth service powers up. We need this file to have permissions that allow users in the bluetooth group to execute it. Currently, the only way I've found out how to automate this process is to use a systemd job as follows.

Create file __/etc/systemd/system/var-run-sdp.path__ with the following content.
`[Unit]
Descrption=Monitor /var/run/sdp

[Install]
WantedBy=bluetooth.service

[Path]
PathExists=/var/run/sdp
Unit=var-run-sdp.service`

Create another file, __/etc/systemd/system/var-run-sdp.service__ with the following content.
`[Unit]
Description=Set permission of /var/run/sdp

[Install]
RequiredBy=var-run-sdp.path

[Service]
Type=simple
ExecStart=/bin/chgrp bluetooth /var/run/sdp`

## 5. Enable and start the job
Use systemctl to start it all up.
`sudo systemctl daemon-reload`
`sudo systemctl enable var-run-sdp.path`
`sudo systemctl enable var-run-sdp.service`
`sudo systemctl start var-run-sdp.path`

### 5.1 Reboot (Optional)
Optionally, reboot the machine.

## 3. Run the SDP server (e.g. Ubuntu 20.04)
`python3 rfcomm_server_sdp.py`

## 4. On the client machine (e.g. beaglebone), run the SDP client:
`python3 rfcomm_client_sdp.py`
