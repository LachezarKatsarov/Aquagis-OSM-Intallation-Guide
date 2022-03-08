# Aquagis OSM Intallation Guide

## 1. Connect to server

In order to start installing the OSM, first we need to connect to the server using a terminal emulator. In our case we are working with PuTTY, but you could use anything that suits your needs. 

   [PuTTY Download Link](https://www.putty.org/)

After installing our terminal emulator, we are connecting to the server using the given IP address and login information. 

## 2. Check Server OS and OS Version 

*This step is important, because some the installation for some of the components could differ depending on the OS Version. 

In order to check this information we run the following command

```
cat /etc/os-release
```

This will show us the information that we are looking for:

 ![Image#1](/src/img/IMG_001.png)  

## 3. NGINX Installation

Installing NGINX is really simple and requires only running the following commands:

```
sudo apt update
```
```
sudo apt install nginx
```

After installation is complete we need to make sure that nginx is running. We can check that when visiting the IP adress of the server with a browser. It should return a page like this: 

![Image#2](/src/img/IMG_002.png)  