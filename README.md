# Nexus-Repository

## Prepare System
- Multiple VM Ubuntu version 20.04 on Hyper-V   [set up hyper-v on windows10](https://github.com/EknarongAphiphutthikul/Install-Hyper-V)
- DNS Server  [set up](https://github.com/EknarongAphiphutthikul/Install-Dns-bind9)
- Update Package On Ubuntu 20.04
  ```sh
  sudo apt-get update
  ```
- Show hostname
  ```sh
  hostnamectl
  ```
- Set hostname
  ```sh
  sudo hostnamectl set-hostname nexus.ake.com
  ```
- Show ip
  ```sh
  ifconfig
  ```
- Set ipv4 internal network (vEthernet-Internal-ME)
  - On cloud : you'll need to disable.
    ```sh
    sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
    ```
    ```console
    network: {config: disabled}
    ```
  - Show file config in netplan
    ```sh
    ls /etc/netplan/
    ```
    ```console
    00-installer-config.yaml
    ```
  - Edit file config in netplan
    ```sh
    sudo nano /etc/netplan/00-installer-config.yaml
    ```
    ```console
    network:
      ethernets:
        eth0:
          dhcp4: false
          addresses:
            -  169.254.19.101/16
          nameservers:
            search: [ ake.com ]
            addresses:
              - 169.254.19.105
        eth1:
          dhcp4: true
      version: 2
    ```
  - Apply Config
    ```sh
    sudo netplan apply
    ```

- Set DNS (change default 127.0.0.53 to 169.254.19.105)  
  > **Important** : Workaround for  [Bug #1624320](https://bugs.launchpad.net/ubuntu/+source/systemd/+bug/1624320)
  ```sh
  sudo rm -f /etc/resolv.conf
  sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
  sudo reboot
  ```

- Install Docker  
  https://github.com/EknarongAphiphutthikul/Install-Docker

- Install Local Persist Volume Plugin  
  - https://unix.stackexchange.com/questions/439106/docker-create-a-persistent-volume-in-a-specific-directory  
  - https://github.com/MatchbookLab/local-persist  
  - https://stackoverflow.com/questions/63227362/docker-volumes-create-options-driver
  ```sh
  curl -fsSL https://raw.githubusercontent.com/MatchbookLab/local-persist/master/scripts/install.sh | sudo bash
  ```
- Create Volume
  ```sh
  mkdir -p /home/akeadm/nexus/data

  docker volume create -d local-persist --opt mountpoint=/home/akeadm/nexus/data --name nexus-data-volume
  ```
----

<br/>

## Install Sonatype Nexus Repository
- Command
  ```sh
  docker run -d --restart unless-stopped -p 8081:8081 --name nexus -v nexus-data-volume:/nexus-data sonatype/nexus3:3.31.1

  docker ps
  ```

- Console
  ```console
  CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS          PORTS                                       NAMES
  a319c71c09fe   sonatype/nexus3:3.31.1   "sh -c ${SONATYPE_DI…"   58 seconds ago   Up 57 seconds   0.0.0.0:8081->8081/tcp, :::8081->8081/tcp   nexus
  ```

- Enable firewall
  ```sh
  sudo ufw allow 8081/tcp
  sudo ufw reload
  sudo ufw status
  ```

- Accessing the User Interface  
  http://nexus.ake.com:8081/

- Administrator password
  ```sh
   docker exec nexus  more /nexus-data/admin.password
  ```