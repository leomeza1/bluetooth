# Service Discovery Protocol (SDP)

Note: python-bluez3 is basically using BlueZ 4 and I have BlueZ 5.

0. Not 100% sure if this is needed. TODO: find out.
sudo sdptool add SP

1. Make sure <user> is in the bluetooth group:
cat /etc/group | grep bluetooth

1.1 If it isn't, then add <user> to the bluetooth group:
sudo usermod -G bluetooth -a <user>

2. Change group of the /var/run/sdp file:
sudo chgrp bluetooth /var/run/sdp

3. To make the change persistent after reboot:

3.1. Create file /etc/systemd/system/var-run-sdp.path with the following content:
[Unit]
Descrption=Monitor /var/run/sdp

[Install]
WantedBy=bluetooth.service

[Path]
PathExists=/var/run/sdp
Unit=var-run-sdp.service

3.2. And another file, /etc/systemd/system/var-run-sdp.service:
[Unit]
Description=Set permission of /var/run/sdp

[Install]
RequiredBy=var-run-sdp.path

[Service]
Type=simple
ExecStart=/bin/chgrp bluetooth /var/run/sdp

3.3. Finally, start it all up:
sudo systemctl daemon-reload
sudo systemctl enable var-run-sdp.path
sudo systemctl enable var-run-sdp.service
sudo systemctl start var-run-sdp.path

3.3.1 Optionally, reboot the system.

4. Run the SDP server
python3 rfcomm_server_sdp.py

