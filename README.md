# Raspberry Pi Cluster — Progress & Tasks

**Started:** yesterday
**Hardware:** 6 x Raspberry Pi 4 → 1 head node + 5 compute nodes (`rpi1`–`rpi5`)
**Guide:** [How to build a Raspberry Pi cluster](https://www.raspberrypi.com/tutorials/cluster-raspberry-pi-tutorial/) (the guide uses 8 nodes, we use 6)

---

## Done so far

- [x] Flashed Raspberry Pi OS Lite onto all 6 SD cards
- [x] Picked one Pi as the head node and set it up (hostname, user, SSH, Wi-Fi)
- [x] Can SSH into the head node
- [x] Set up Git on the head node

---

## To do

### 1. Wiring

- [x] Mount/place all 6 Pis and the switch
- [x] Run an Ethernet cable from each Pi to the switch
- [x] Sort out power for all 6 Pis (PoE HATs, or individual power supplies)
- [ ] Plug the USB→Ethernet adapter into the head node and connect it to the normal network
- [ ] Plug the SSD into the head node via USB 3

### 2. Head node — networking

- [x] Give the onboard port (`eth0`) a static IP: `192.168.50.1`
- [ ] Check the outside-world port (`eth1`) gets an IP from the normal network
- [x] Install `isc-dhcp-server`
- [x] Configure `/etc/dhcp/dhcpd.conf` for the `192.168.50.0/24` cluster network
- [x] Set `INTERFACESv4="eth0"` in `/etc/default/isc-dhcp-server`
- [ ] Add cluster hostnames to `/etc/hosts`
- [ ] Reboot and verify with `dhcp-lease-list`

### 3. Head node — storage

- [ ] Partition and format the SSD as ext4
- [ ] Mount it at `/mnt/usb` and add it to `/etc/fstab`
- [ ] Install `nfs-kernel-server`
- [ ] Create and share `/mnt/usb/scratch` over NFS
- [ ] Enable and start `rpcbind` + `nfs-server`

### 4. Head node — boot server

- [ ] Install `tftpd-hpa` and `kpartx`
- [ ] Set up `/mnt/usb/tftpboot` and configure `/etc/default/tftpd-hpa`
- [ ] Download the OS image and build the first node image at `/mnt/usb/rpi1`
- [ ] Export the image over NFS

### 5. Compute node 1 (`rpi1`)

- [ ] Boot it from SD, SSH in
- [ ] Enable network boot via `raspi-config` → Advanced Options → Boot Order
- [ ] Verify `BOOT_ORDER=0xf21` with `vcgencmd bootloader_config`
- [ ] Write down its MAC address and serial number
- [ ] Add a `host rpi1` entry to `dhcpd.conf`
- [ ] Remove the SD card and boot it over the network
- [ ] Set hostname to `rpi1`
- [ ] Disable `resize2fs_once` and `sshswitch`, remove `dphys-swapfile`
- [ ] Mount `/scratch` from the head node

### 6. Compute nodes 2–5

Repeat the `rpi1` process for each: note MAC + serial, copy the image, add a `dhcpd.conf`
entry, update `cmdline.txt` and `/etc/hostname`, then network boot.

- [ ] `rpi2`
- [ ] `rpi3`
- [ ] `rpi4`
- [ ] `rpi5`

### 7. Finishing up

- [ ] Set up passwordless SSH from head node to all 5 compute nodes
- [ ] Enable IP forwarding + iptables NAT so compute nodes reach the internet
- [ ] Install `pssh` and create `.pssh_hosts` with `rpi1`–`rpi5`
- [ ] Test with `parallel-ssh -i -h .pssh_hosts free -h`
- [ ] Commit all config files to Git

### 8. Later / decide as a team

- [ ] Pick what the cluster actually runs (MPI, Kubernetes, Docker Swarm, etc.)
- [ ] Write a clean shutdown script for the whole cluster
- [ ] Write a README in the Git repo

---

## Node table

Fill this in as you go — you will need it repeatedly.

| Hostname | Role    | IP              | MAC address | Serial number |
| -------- | ------- | --------------- | ----------- | ------------- |
| cluster  | head    | `192.168.50.1`  |             |               |
| switch   | switch  | `192.168.50.254`|             |               |
| rpi1     | compute | `192.168.50.11` |             |               |
| rpi2     | compute | `192.168.50.12` |             |               |
| rpi3     | compute | `192.168.50.13` |             |               |
| rpi4     | compute | `192.168.50.14` |             |               |
| rpi5     | compute | `192.168.50.15` |             |               |

---

## Notes

- Write down each Pi's MAC address and serial number **before** removing its SD card.
- The tutorial goes up to `rpi7`. With 6 Pis we only need `rpi1`–`rpi5`.
- Compute nodes should **not** have Wi-Fi configured — only the head node.
- Adding the SSD to `/etc/fstab` can hang boot if the disk is missing. Always test a manual
  mount first.
