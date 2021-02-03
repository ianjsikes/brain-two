# rpi-plex-mk-ii

- Use [Raspberry Pi Imager][1] to format SD card, then flash Raspbian
- Setup rpi ssh, password, etc.
- (Optional) Hook up Power switch to avoid corrupting filesystem
  - Add this line to `/boot/config.txt` on the SD card: `dtoverlay=gpio-shutdown`
  - Hook up a momentary switch to connect pins 5 and 6
- Change rpi password for user `pi`: `mambomouth420`
- Verify that SSH works
  - Run `arp -a` to list device IPs on local network to find pi
  - Run `ssh pi@IPADDRESS` and enter password
- Update rpi software
  - sudo apt-get update && sudo apt-get upgrade
- Setup external drive and samba file share
  - Follow this guide: https://lefkowitz.me/setup-a-network-share-via-samba-on-your-raspberry-pi/
    - Mount at `/mnt/mortimer`
  - Run `sudo smbpasswd -a pi` to set samba password
  - Connect in Finder with `smb://IPADDRESS`, `pi`, `mambomouth420`
- Install plex
  - Follow this guide: https://pimylifeup.com/raspberry-pi-plex-server/
  - Login to plex web and claim server
- Install docker and docker-compose: https://dev.to/rohansawant/installing-docker-and-docker-compose-on-the-raspberry-pi-in-5-simple-steps-3mgl
  - Create a `scripts` directory
  - Create a Dockerfile that extends haugene/transmission-openvpn and installs filebot
  - Create a docker-compose.yml that runs the container, configured to run transmission-postprocess


---

## How to backup SD card
- Shutdown rpi and remove SD card. connect to mac
- Run `diskutil list` to find the disk. add an `r` to the name like `/dev/disk2` -> `/dev/rdisk2`
- Run this command `sudo gdd if=/dev/rdisk2 of=YYYY-MM-DD-note.dmg status=progress bs=4M`

- OR run `pi-backup some-note`

## How to restore from backup
- Unmount the disk: `sudo diskutil unmountDisk /dev/disk2`
- `sudo gdd of=/dev/rdisk2 if=YYYY-MM-DD-note.dmg status=progress bs=4M`

[1]: https://www.raspberrypi.org/software/