# EthereumNodePi
Running a full Ethereum on your raspberry pi

## Buying the required part
* Raspberry Pi 3B+ — this is the latest full model, which has wifi and bluetooth built in.
* An External Hard Drive— 500GB to 1TB
* A quality case and heatsinks 
* 32G+ microSD card — We’re storing the chain data on the hard drive, so only need enough memory to run the operating system.

## Installation
### Install Raspbian

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
### Install Docker

debian buster are not avalable yet but there is a workaround:
 ```
curl -sL get.docker.com | sed 's/9)/10)/' | sh
 ```

due to an issue in containerd used in official package, dont install like this!
 ```
sudo apt-get install docker.io docker-compose
 ```
 
Store containers in an external USB drive. Change the following line
 ```
sudo vi /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -g /mnt/ethereum-data/ /media/USBdrive/docker -H fd://
 ```

Reload changes
 ```
sudo systemctl daemon-reload
sudo systemctl restart docker
 ```

You can check that it worked with
 ```
sudo docker info | grep Root
Docker Root Dir: /media/USBdrive/docker
```



### References
 * https://www.raspberrypi.org/documentation/configuration/external-storage.md
 
 
 
 

