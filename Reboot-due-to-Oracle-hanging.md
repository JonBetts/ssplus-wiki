When Oracle prevents access even to the `oracle` user trying `sqlplus "/ AS SYSDBA"`, a reboot is normally required to reset everything. For this to be timely, the problem processes need to be force-killed like so:
```bash
sudo /etc/init.d/bbc stop
sudo killall -9 oracle rman sqlplus
sudo reboot
```