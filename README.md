
# Plex media server setup + Arrs setup

This setup can be done on different setups and operating systems. For this setup, we will use Ubuntu Server along with CasaOS as it is a very easy to use setup


## 1. Preparation

- You will need a at least __8GB USB Stick__. First, download Ubuntu Server

```bash
  https://ubuntu.com/download/server
```

- Get a tool like __balenaEtcher__ or __Rufus__ to flash the image onto the USB stick

```bash
  https://etcher.balena.io/
```

- Flash the linux image onto the disk and wait for it to complete (__save any data on it before since it will be formatted__)
## 2. Ubuntu Server Setup

- Pretty simple installation, you can follow the steps mentioned in the link below

- Some additional notes:
  - Highly suggested to use an ethernet cable connection instead of wifi
  - Use an entire disk for storage, best to use the internal memory of the Pi/PC/Laptop/etc
  - No need for partitioning, default settings are ok
  - Make sure to check the __install SSH server__ option
  - No need to install any other packages here, we will add what we need later

```bash
  https://ubuntu.com/tutorials/install-ubuntu-server
```

- After the installation is complete, you can now ssh from another machine directly into the server

```bash
  ssh your-username@server-ip-address
```

- Do a system update

```bash
  sudo apt update
```
```bash
  sudo apt upgrade
```

- Restart the machine after the update has finished
## 3. CasaOS Installation

- Pretty simple installation, use the following command to start the installation

```bash
  sudo curl -fsSL https://get.casaos.io | sudo bash
```

- After it is installed, you can access the web interface from anywhere inside the network by entering the server ip in any browser
- If you don't know the IP address, go into the terminal of the machine and use this

```bash
  ip a
```

- The IP can be found under the inet property in the network interface used
## 4. External Drive Mounting (optional)
Follow these steps if you want to use an external hard drive / ssd for you media storage.
The reason I chose to do this is because in some cases linux will start renaming and partitioning your external hard drives. In this way, we can create a permanent mounting point that will always be used

- Check if the drive is already mounted. It should be something like sda, sdb, etc. Each one will have names like sda1, sda2, etc for each partition on that drive

```bash
  lsblk
```

- Format the drive into a format like exfat. We will need a package for that

```bash
  sudo apt update
```
```bash
  sudo apt install exfatprogs -y
```

- Check what drives are mounted

```bash
  df -h
```

- Unmount the drive you want to format. Replace sda1 with the name of you partitioned as seen from df -h

```bash
  sudo umount /dev/sda1
```

- Open the fdisk utility

```bash
  sudo fdisk /dev/sda
```

- Now follow the next steps:
  - Type __d__ until there are no more partitions
  - Type __n__ to create a new partition and accept all the defaults
  - Type __t__ and select the exfat format (Code 11 from the list usually, but check the full list to be sure of the code you need)
  - Type __w__ to write changes and exit

- Verify the formatting. You should see something like this: __/dev/sda1: UUID="xxxx-xxxx" TYPE="exfat"__

```bash
  sudo blkid /dev/sda1
```

- Create the mounting point

```bash
  sudo mkdir -p /mnt/media1
```

- Mount the drive

```bash
  sudo mount /dev/sda1 /mnt/media1
```

- Check the drive

```bash
  ls /mnt/media1
```

- Now we will enable automatic mounting. First, get the UUID

```bash
  sudo blkid /dev/sda1
```

- Edit the __/etc/fstab__ file

```bash
  sudo nano /etc/fstab
```

- Add the following line (replace xxxx-xxxx with your UUID)

```bash
  UUID=xxxx-xxxx /mnt/media1 exfat defaults 0 2
```

- Test the configuration

```bash
  sudo mount -a
```

- Rename the drive (optional)

```bash
  sudo exfatlabel /dev/sda1 "NewLabel"
```


## 5. Setup Plex Server

- From the CasaOS App Store, search for Plex (the one with Media label, not BigBearCasaOS)
- The app should now appear on the Home screen. Click the 3 dots from the top right corner to configure the docker image
- One thing you need to configure is to add the Volumes containing the media so that docker can see them. Example:
Host - the actual path to the drive
```bash
  /mnt/Expansion2/Movies
```
COntainer - the one you will see inside the image
```bash
  /Movies
```
- Add the remaining drives according to your needs (TV Shows, Music, etc)
- Now you can save the configuration and the image will rebuild automatically
- Enter the Plex App and start setting up the server by following the steps on screen. This is pretty simple to setup so I will only mention some tips:
  - Enable automatic media folders scan so that each time you add a file Plex will auto add it to the home page
  - When setting up the media libraries, multiple folders can be selected for each library (example: Movies can be read from multiple folders, even from different drives)
  - The default home page of Plex is loaded with stuff like Live TV, Discover, etc. You can unpin anything you like
  - Plex Pass is optional, you will not need it to run your server or to watch your media, however, it has some very nice features, the most important one being __Enable Hardware Acceleration__

- Once the server is up and running, you can access it from any device you like, since Plex offers support for mostly all platforms (Windows, Linux, Android/IOS, TV, etc). Do note that for mobile streaming you will need a Plex Pass, which will also allow you to download media for offline view
## 6. Static IP + Port Forwarding for Plex

These steps will differ from each person to another as each router manufacturer has a different interface, but the principles here will apply for all of them. Plex already offers the option to access the media from outside the network using UPnP, however this can sometimes fail, so we will forward the port manually.

- First, we need to setup a static ip address for the Ubuntu Server device. Get the MAC address of the device by opening the terminal in the machine and typing
```bash
  ip a
```
Find the network interface that your server uses and the MAC Address should be in the link/ether section. It will be in a format like this
```bash
  XX:XX:XX:XX:XX:XX
```
- Go to the home page of your router and access
```bash
  LAN/DHCP Static IP Configuration
```
- Click on New: 
  - For the MAC Address, type the one you found above
  - For IP Address, type in the IP of you local network and replace the final digits with something you want
  For example, if your network IP is something like 192.168.100.XX, replace XX to get an IP like
  ```bash
  192.168.100.66
  ```
  Avoid using reserved IP addresses like 1 or 255

- Now, for port forwarding, go to
```bash
  Forward Rules/Port Mapping Configuration
  ```
- Click on New:
  - Internal Host sould be you linux server, which will have the static IP we set-up above
  - Protocol should be set to TCP/UDP
  - Internal port number can be something like 32400 - 32400 (The default port Plex uses but this can be changed)
  - External port number can be also set to 32400 - 32400
  - External Source IP Address and External source port number should be left blank
  Note that the port can be configured as you like, but this setup is pretty simple to avoid confusion

- Now we are done configuring the Router so you can close the page

- To be sure you Plex server can be accessed from outside the local network:
```bash
  Go into Plex > Settings > Remote Access > Check Manually specify public port and type in 32400 (or the port you configured above)
```
Click on Apply and if everything has been setup correctly you should see a message in green like
```bash
  Fully accessible outside your network
```
To further test this, open Plex on a device that is not connected on the local network (A phone with mobile data on and WIFI off) and check if everything is available to you.
## 7. qBitTorrent Setup + Configuration

- Go to the CasaOS App Store, search for qBitTorrent and install it
- Click the 3 dots to configure the image
  - For Volumes, add the external drive you want your media to be downloaded to
   Host
  ```bash
  /mnt/Expansion2
  ```
  Container
  ```bash
  /Expansion2
  ```
  __Please remember this mapping for later when we setup Sonarr and Radarr because it will be very important__
  - For PGID/PUID, I set 0 for both of them as this is the ID of the root user
  - You can also tick the Privileges toggle
  - Restart Policy > Always

- You can now save the configuration and open the App. The default port should be __8181__. If you will see just a black page with an Unauthorized message, simply click on the link and hit enter again, you should see the Login page then.
- The default user is Admin, and the password can be found as follows:
```bash
  Open CasaOS homepage in a new tab > click to configure qBitTorrent > Up top there should be a Terminal and Logs button > Logs
  ```
  You should see the password there

- Once logged in, go into Settings by clicking the Cog Wheel on the top bar

  - Downloads Tab
    - Default Save Path: _/Expansion2/downloads/complete_ (or your preffered folder as you have it set up)
  - BitTorrent Tab
    - Max active checking torrents: 1
    - Maximum active downloads: 1
    - For Seeding limits, check When ratio reaches: 1 and When total seeding time reaches: 0
  - WebUI Tab
    - Ip Address: * and Port: 8080
    - Authentication: here you can change the password so you won't need a temporary one anymore
    - Enable Bypass authentication for clients on localhost
  - Advanced Tab
    - .torrent file size limit: 100 (this can be customized to your prefferences)

- Restart the docker image and we are done.
## 8. Sonarr & Radarr Setup + Configuration
Sonarr will be used to search for the TV Shows and Radarr for Movies. I will go over installing and setting up Radarr, but the exact same steps apply for Sonarr aswell

- Go to the CasaOS App Store and search for Radarr. Install it
- Go to the 3 dots to configure it:
  - For Volumes:
  Host
  ```bash
  /mnt/Expansion2
  ```
  Container
  ```bash
  /Expansion2
  ```
  __Make sure you set this up exactly as we set up qBitTorrent previously__
  - PGID and PUID should be set to 0
  - Enable Privileges
  - Restart Policy: Always

- Now open up Radarr and let's configure it
  - Go to Settings > Media Management
    - Enable Advanced Settings from the Top
    - (Optional) Enable Rename Movies
    - (Optional) Enable Replace Illegal Characters
    - Enable Use Hardlinks instead of Copy (__this is very important__)
    - Scroll down and add a Root Folder for /Expansion2/Movies (or the path you want the movies to be stored to)
  - Go to Settings > Download Clients
    - Add qBitTorrent and configure it:
      - Click Enable
      - Host: ip address of your server: example 192.168.100.66
      - Port: 8181
      - Username and Password: the ones you setup in qBitTorrent settings
      - Enable Remove Completed
      - Click Test and if everything is fine you can Save
    - Enable Automatically import completed downloads from download Clients
    - Check For Finished Downloads Interval: 1 minute
    - Enable Redownload Failed
    - Enable Redownload Failed from Interactive Search
    - Add a Remove Path Mapping:

      Host
      ```bash
      192.168.100.66
      ```
      Remote Path
      ```bash
      /Expansion2/downloads/complete/
      ```
      Local Path
      ```bash
      /Expansion2/downloads/complete/
      ```
      Change these settings according to your mappings

- Change other settings according to you prefferences, but keep the ones configured above as they are. Radarr is now ready and you can follow the exact same steps for Sonarr
## 9. Prowlarr Setup + Configuration
We will use Prowlarr to setup the Indexers Sonarr and Radarr will then use, so it is a way to centralize this setup instead of setting up Indexes for each of them

- Open CasaOS App Store and search for Prowlarr. Once installed, click the 3 dots to configure it
  - PGID and PUID can be set to 1000
  - Restart Policy: Always
- Open Prowlarr and let's configure it:
  - Go to Settings > Apps. Here we will link Sonarr and Radarr
  - Add Radarr and configure it as follows:
    - Sync Level: Full Sync
    - Prowlarr Server: http://192.168.100.66:9696 (configure it with your server's IP)
    - Radarr Server: http://192.168.100.66:7878 (configure it with your server's IP)
    - API Key: To get this, go into the Radarr App > Settings > General > API Key
    - Click on Test and if everything is fine you can Save
  - Add Sonarr aswell and configure it the same (change the Sonarr port to 8989 or the port Sonarr runs on your server)
  - Go to Indexers at the top and add a new Indexer. For private ones, the settings will most likely differ, but I will configure FileList as an example:
    - Base URL: https://filelist.io/
    - Username: You FileList Username
    - Passkey: to get this, go to Filelist > Profile > Scroll until you see Reset Passkey and copy it from there
    - Click on Test and if everything is fine you can Save
    - You can add any number of indexers you want, some good public ones are IsoHunt and BitSearch
## 10. (Optional - Recommended) Overseer Setup & Configuration
Overseer is a very useful application that will link to Sonarr and Radarr and allow you searching for TV Shows/Movies in each individual app. It will also allow you to setup a request based system inside you media server where people who have access to that Plex Server will have a way to request Movies/TV Shows, which you can later approve/deny.

- Open CasaOS App Store and search for Overseer
- Click the 3 dots to configure the image:
  - Add Volume
  Host
  ```bash
  /mnt/Expansion2
  ```

  Container
  ```bash
  /Expansion2
  ```
  - PGID and PUID can be set to 0
  - Restart Policy: Always
- Install it and we'll start configuring it:
  - Go to Settings > General
    - Enable Allow Partial Series Requests
  - Go to Settings > Users
    - Enable Local Sign-In
    - Enable New Plex Sign-In
  - Go to Settings > Plex
    - Hostname or IP Address: the IP address of the server (example 192.168.100.66)
    - Port: the port on which Plex is running (by default is 32400)
    - Click on Sync Libraries and if setup correctly you will see your Libraries (which you should enable for syncing)
  - Go to Settings > Services
    - Add the Radarr app here and configure it:
      - Hostname or IP Address: the server IP (example 192.168.100.66)
      - Port: the port Radarr is running on (by default 7878)
      - APY Key: to find this, go into the Radarr app > Setting > General > API Key
      - Quality Profile: set this to whatever you preffer (you can always change the profile when requesting)
      - Root Folder: should be the folder Radarr uses for Movies (example /Expansion2/Movies)
      - Enable Enable Automatic Search
      - Follow the exact steps to add Sonarr (change the port to the one Sonarr uses and the root folder to the TV Shows)

- You can customize the settings as you like, but try to keep the ones configured above as they are.
- To test if everything is working, go to the Discover tab and search for any Movie/TV Show. Click on request and they now should appear in either Sonarr/Radarr as downloading. Additionally, you can open qBitTorrent aswell to check if the download has been started. As an Overseer Admin, once you request a Movie/TV Show it will not need an approval and will automatically start.
- Overseer can also be configured to send notifications in different ways like mail/discord/etc.
## 11. Conclusion & Additional Tips

- There are multiple ways to setup a Media Server, the steps presented here are just the ones I have done for my own setup and they work without issues.
- It is highly recommended to also install Portainer from the CasaOS App Store, as it is a simple install with no other configuration. CasaOS will install all the apps as Docker images, so having something like Portainer is very useful to see and manage all your images
- For all of my Docker images I have a Always Restart policy, so in case the server restarts, they can automatically start
- You can configure you server machine to power on automatically in case power goes off. When it goes back up your server will start without you needing to manually power it on
