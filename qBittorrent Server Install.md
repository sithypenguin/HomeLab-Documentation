# qBittorrent Server Install w/ WebUI

## Foreward

The purpose of this installation was so that a user could open a webUI from anywhere on the local network, paste in the link/magnet for a torrent, and then download that file to a location a network attached storage (NAS) device. The server this is running on is an Ubuntu 22.04.01 VM running on Proxmox. Some of the steps are not required for the actual functionality of the server, only for easier management and housekeeping. The steps that are not required are noted as such. I am writing this as practice for writing better documentation for users of the lowest skill level and to pass on knowledge that the community has unknowingly shared to me via various forum posts and threads. There will be explanations on each command to the best of my understanding. Any corrections or clarifications are welcome and invited. I only want to help and be helpful as best as I can.

**!! Disclaimer !!**

This guide is intended to setup a torrent server for the legal download of torrents for things like linux distros. This is not intended to be used as a way of downloading any copyrighted materials.

**!! Disclaimer !!**

## Process

1. Ensure that your OS is updated and upgraded
> sudo apt-get update && sudo apt-get upgrade

- This command will simultaneously update and upgrade your debian based distrobution or anything using the apt package mananger.

2. Install cifs-utils and linux generic packages
>sudo apt-get install cifs-utils linux-generic

- This command allows the use of cifs or SAMBA/Windows shares on linux. The linux-generic package is installed so that the "iocharset" argument can be used


##### *Not Required* 

Install  net-tools and neofetch 

>sudo apt-get install net-tools neofetch 

- Net-tools allows the use of ifconfig and neofetch is a command line program that displays information about the current machine. Not required for the functionality of this application, just something I like to use for management.

3. Create .smbcreds file and enter credentials 
> nano .smbcreds 

- Nano is a text editor that is normally installed on most linux distros. I am new-ish so I am used to it and find it easy to use. ".smbcreds" is the file name in this case. You can make this whatever you want. There is probably a more secure way of doing this, but considering this isn't public facing it wasn't the highest thing on my list. By default this will most likely be created in the home directory of the user currently signed in.

4. Create a directory for the mount
> mkdir HomeNAS

- mkdir is the command for making directories. This will create a directory in the current directory that you are in. "HomeNAS" being the name of the folder, you can name this whatever you want, just ensure you note it down as you will need it later.

5. Change fstab to mount automagically
> sudo nano /etc/fstab

- This command should open up the fstab file within nano so that is can be edited. From here you can put in this command at the bottom of the file. Once complete you can save and close out of the file. Below is an explanation of all of the components of the line. Noticed after the "credentials...smbcreds" "," we are putting a comma after all of the other options, and **no spaces**

> //ip_of_server/file/path /home/username/folder cifs credentials=/username/.smbcreds,rw,user,nofail,iocharset=utf8 0 0

 - **//ip_of_server/file/path**: This item in this case describes the remote filesystem to be mounted. In reality this may look like this //192.168.10.2/Reader/Downloads
 - **/home/username/folder**: This item in this case describes where in the local system, will the previous filesystem be mounted to. Think of it like a literal placeholder for the remote filesystem. In reality this may look something like: /home/sithypenguin/HomeNAS
 - **cifs**: This describes the filesystem to be mounted or maybe more accurately what "language" to speak to the filesystem. 
 - **credentials=/username/.smbcreds**: Instead of listing the credentials that are being used to communicate with the NAS, we are referring back to the smbcreds file that was made in Step 3.
 - **rw**: We are stating this should be considered read/write enabled
 - **user**: This command allows a standard **non-sudo** user to mount the drive
 - **nofail**: This command doesn't report any errors in the event of a failure. You can omit this should you want to see the errors. 
 - **iocharset=utf8**: Character set to use for converting between 8 bit characters and 16 bit Unicode characters.

 - **0 0**: At the very end of the string there are two 0's The first "0" indicates which filesystems need to be dumped. In this context, 0 means "don't dump." The second 0 is used to determine the order in which filesystem checks are done at boot time. This 0 means "don't check the filesystem. It should be stated that these are default values if nothing is present in their places, but leaving them in, helped me understand it better.

- With this inplace you should now be able to save the file and close it. 


6. Mount the NAS *without* sudo
> mount -a

- This command will allow you to mount everything within the /etc/fstab file

7. Install the qbittorrent-nox package. 

> sudo apt install qbittorrent-nox 

- This package is the point of this documentation. This is the package that runs the torrent server. The **-nox** piece is what installs the web browser portion.

8a. Start qbittorrent

> qbittorrent-nox

- This will start the service within the terminal window but as a process which means you would have to leave the terminal window open. We want to run this as a service so its running in the background, not the foreground

8b. Run qbittorrent as a service
> sudo nano /etc/systemd/system/qbittorrent.service
> [Unit] 
    Description=qBittorrent-nox service 
    Wants=network-online.target 
    After=network-online.target nss-lookup.target 

    [Service] 
    Type=exec 
    Restart=always 
    RestartSec=1 
    User=pi 
    UMask=0000 
    ExecStart=/usr/bin/qbittorrent-nox 

    [Install] 
    WantedBy=multi-user.target 

- Open/create the service file. Within that file enter the information above. Change the **User=** from pi to whatever the user is in your case

9. Update the service manager to see the service that was just made

> sudo systemctl daemon-reload

- This allows that systemctl daemon to see the new service file that was just created and use it

10. Start the service

> sudo systemctl start qbittorrent

- Starts the service so the terminal window can be closed down

11. Confirm the service has started

> sudo systemctl status qbittorrent

- This will return the status of the service and if it is running or not. If it failed to run check the service file from step 8b and make sure there are no typos. 

12. Enable the service to run on startup 

> sudo systemctl enable qbittorrent

- Enables the service to run on startup so you don't need to run the service everytime manually after a reboot

13. First Login

> 192.168.10.10:**8080***
> Username: admin
> password: adminadmin 

 - On a web browser (localhost or another computer on the same network) type in the IP address of the server followed by the port. The password is admin twice with no space. Not a typo. I had to check my notes and online to be sure I didn't mess it up


 ### At this point your qbittorrent server is setup and ready to go!

 