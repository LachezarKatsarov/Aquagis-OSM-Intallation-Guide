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

### 3.1 NGINX Settings

In order for nginx to work the way we want it, we need to change a few of its settings. First we need to locate its default file, which should be in the following directory:

```
cd /etc/nginx/sites-available
```

Then we are going to go in the editor for it:

```
sudo nano default
```

In this file there are a few things that need to be changed. Once opened in the editior the file should look like this:

![Image#3](/src/img/IMG_003.png)  

And after our changes it should be like this:

![Image#4](/src/img/IMG_004.png)  


Here are the changes to copy:

```
root /srv/www/tiles;
```
```
location / {
      # First attempt to serve request as file, then
      # as directory, then fall back to displaying a 404.
      try_files $uri $uri/ index.html;
}
```
```
location /iD/ {
      alias /var/www/iD/;
}

```

After we have saved those changes, we must restart the nginx for them to work:

```
sudo systemctl restart nginx
```

## 4. Clone Aquagis-Backend repository on the server

The next step in our installation is to get the backend up and running on the server. 

The repository for the backend is accesible on the following [link](https://github.com/aqua-gis/aquagis-backend)

We need to clone it in this directory on our server: 

```
cd /srv/apps/
```
```
sudo git clone https://github.com/aqua-gis/aquagis-backend.git
```

## 5. Install PostgreSQL on the server

In order to continue we need to install PostgreSQL on our server. For our application we are installing PostgreSQL v13. According to the server OS and version, we could check how to install it online or follow the following commands, if the system is the same. [LINK](https://computingforgeeks.com/how-to-install-postgresql-13-on-ubuntu/)

### 5.1. Add PostgreSQL 13 repository to Ubuntu 20.04
```
sudo apt -y install vim bash-completion wget

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
```
### 5.2 Install PostgreSQL 13 on Ubuntu 20.04

```
sudo apt update

sudo apt install postgresql-13 postgresql-client-13
```

After the installation is complete we need to change the password of the dafault user that was automatically created. In order to do that we need to switch our account to the postgres one, using this command:

```
sudo su - postgres
```

In order to change the password, we need to generate a random strong one online and use it as the new for the postgres account:

```
psql -c "alter user postgres with password 'nA62Qpy6wGM:@Lr%'"
```

After changing the main password, we continue by creating a new user for the database. For this project we are using **gis** as username and password.

```
createuser --interactive --pwprompt
```

We create the user by entering the username and the password twice adn also we give that user "superuser role".
