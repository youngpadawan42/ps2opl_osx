# How to setup SMB on Mac OSX for PS2 Open Loader
It has been known for long that SMB on Mac OSX doesn't play well with PS2. For a long time a known trick has been to update SMB on Mac OSX using [SMBUp](http://eduo.info/apps/smbup). However the problem with SMBUp is that it doesn't work with OSX  El Capitan and later versions.

This tutorial will show how to setup a SMB server using Docker on your Mac, that will work with PS2 Open Loader.

## Pre-requisite
In order to get started you will need the following:

- Mac with OSX and e.g. High Sierra that will work as SMB server
- [Docker for Mac](https://docs.docker.com/docker-for-mac/) installed and working
- Somewhere on your filesystem, the required [directory structure](http://www.ps2-home.com/forum/app.php/page/opl_folder_structure) for PS2OPL. 

## Stop all filesharing services on Mac
First thing is to stop all filesharing services on Mac.
- Settings -> Sharing, make sure File Sharing is unchecked.
- Stop remaining services that will interfer with running SMB

From a command line, execute the following command to complete stop all SMB related service.
```
sudo launchctl disable system/netbiosd
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.netbiosd.plist
sudo launchctl unload -w /System/Library/LaunchAgents/com.apple.netbiosd.plist
```

# Create a new SMB server using docker
By using an existing image with SMB, we will configure our own. The image settings are found in the `docker-compose.yml`, and an example can be found in this reposity.

# Important settings in docker-compose.yml:

## What to share:
````
    # Shares
    - /Volumes/Data/Playstation2/PS2SMB:/mnt/ps2smb
````
This states that the folder on our Mac `/Volumes/Data/Playstation2/PS2SMB` will be availble in the container and shared for use with PS2 OPL. The `/mnt/ps2smb` is the location inside the container. Replace the `/Volume/...` with the full paht to your own PS2 OPL SMB structure on your Mac. If you want to change the mount point in the container, you also need to modify the `smb.conf` - more on that later.

## User access
````
  environment:
    - USERNAME=joe
    - PASSWORD=samba
````
Should be obvious - you need these info on your PS2 later in order to connect to the SMB server.

# Important settings in smb.conf:
## What to share:
````
[PS2SMB]
        comment = PS2SMB
        path = /mnt/ps2smb
        browseable = yes
        read only = no
        guest ok = yes
        public = yes
        available = yes
````
`[PS2SMB]` the name of the share as seen by the PS2, keep the name as it is.
`path = /mnt/ps2smb` the mount point inside the container, remember how this was defined in the docker-compose.yml file?

# Start our new SMB server
You start the server by running the following command in the same folder as the two config files:
````
docker-compose run samba
````

