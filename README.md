(https://raw.githubusercontent.com/Tokens-Economy/EthereumNodePi/master/EthereumLoveRaspberryPi.png)
# EthereumNodePi
Running an Ethereum light node on your raspberry pi 3B+/4.

Note that the current raspberry pi 3 is not able to run a full node:  Processing power, i/o bandwidth, and network bandwidth, mostly make it impossible to run a full node. Maybe this may have change with the raspberry pi 4 - 4 GB?

## Buying the required part
* Raspberry Pi 3B+ — this is the latest full model, which has wifi and bluetooth built in.
* An External Hard Drive— 500GB to 1TB
* A case and heatsinks 
* 16GB+/32G+ microSD card — We’re storing the chain data on the hard drive, so we only need enough memory to run the operating system.

## Installation
### Install Raspbian
Install Raspbian https://www.raspberrypi.org/downloads/raspbian/ on your sdcard using https://www.balena.io/etcher/
You need to connect the PI first to a HDMI screen, and keyboard
```
sudo raspi-config
```
Activate Wifi, and SSH under Boot options

It is a good idea to update the operating system
```
sudo apt-get update
sudo apt-get upgrade
```

### Connect USB disk
Format to Ext4 and mount the filesystem
 ```
sudo mkfs.ext4 /dev/sda1
sudo mkdir /mnt/ethereum-data
sudo mount /dev/sda1 /mnt/ethereum-data/
 ```
 
 Verify that the storage device is mounted successfully by listing the contents:
```
ls /mnt/ethereum-data/
```
### Setting up automatic USB disk mounting
You can modify the fstab file to define the location where the storage device will be automatically mounted when the Raspberry Pi starts up. In the fstab file, the disk partition is identified by the universally unique identifier (UUID).
 
Get the UUID of the disk partition:
```
sudo blkid
```
Find the disk partition from the list and note the UUID. For example,  bb72cd58-8477-4639-8214-31d85c7b0c5b

Open the fstab file using a command line editor such as nano or vi:
```
sudo vi /etc/fstab
```
Add the following line in the fstab file:
```
UUID=bb72cd58-8477-4639-8214-31d85c7b0c5b /mnt/mydisk ext4 defaults,auto,umask=000,users,rw,nofail 0 0
```

### Install Etheruem

Get the ARM7 (Raspberry Pi 3B+) build from 
https://geth.ethereum.org/downloads/

```
cd /mnt/ethereum-data
wget https://gethstore.blob.core.windows.net/builds/geth-linux-arm7-1.8.27-4bcc0a37.tar.gz
tar xvf geth-linux-arm7-1.8.27-4bcc0a37.tar.gz
mv geth-linux-arm7-1.8.27-4bcc0a37/geth /usr/local/bin/geth
```
  
First if we use geth to create a new account.
```
geth account new
```

We want ethereum to start at boot time
```
sudo vi /etc/systemd/system/geth@.service
```
and enter 
```
[Unit]
Description=Ethereum daemon
Requires=network.target

[Service]
Type=simple
User=%I
ExecStart=/usr/local/bin/geth --syncmode light --cache 64 --maxpeers 12 -datadir=/mnt/ethereum-data/
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
As per geth help document there are 3 ways you can do sync to network:

* `--syncmode full`: Geth client will download Block header + Block data + full Validation: Is called eth full node
* `--syncmode fast`: Geth client will download Block header + Block data + validate for last 1k transactions.
* `--syncmode light`: Geth client will download Current state + Asks nodes for as its need. Light node It will request missing blocks from full nodes

now enable the service and start it
```
sudo systemctl enable geth@pi.service
sudo systemctl start geth@pi.service
```

Youre done!
You can now 

```
get attach
admin.peers
eth.syncing
```
to see the number of peer attached to your node or the status
