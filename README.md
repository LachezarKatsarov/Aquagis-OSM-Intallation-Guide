# Ръководство за инсталиране на Open Street Map (OSM) и стартиране на iD Editor

## 1. Свързване към сървъра

За да започнем инсталацията на OSM, първо трябва да се вържем към сървъра използвайки терминално приложение (емулатор). В случая и за целите на това ръководство ще работим с PuTTY, но може да бъде използвано и друго приложение по ваш избор.

   [Линк за изтегляне на PuTTY](https://www.putty.org/)

След като инсталираме успешно емулатора, се връзваме към сървара използвайки предоставеното ни IP и данни за потребител и парола.


## 2. Проверка на вид и версия на операционна система

**!!!Тази стъпка е важна, защото инсталацията на част от компонентите се различава за различните операционни системи и версии**

За да проверим вида операционна система под която работи сървъра, изпълняваме следната команда:

```
cat /etc/os-release
```
Това ще ни покаже търсената от нас информация: 

 ![Image#1](/src/img/IMG_001.png)  

## 3. Инсталиране на NGINX

Installing NGINX is really simple and requires only running the following commands:

Инсталирането на NGINX е много бърз и лесен процес, необходимо е само да изпълним следните команди в терминала:

```
sudo apt update
```
```
sudo apt install nginx
```

След като иснталацията завърши успешно е необходимо да проверим, че NGINX работи. Има два варианта да направим това:

- Вариант 1: Отваряме браузъра и изписваме IP адреса на сървъра, като при работещ NGINX това би трябвало да зареди следната страница: 

![Image#2](/src/img/IMG_002.png)  

- Вариант 2: Проверяваме статуса на NGINX чрез терминала със следната команда:

```
systemctl status nginx
```

![Image#2.1](/src/img/IMG_002.1.png)  

!!!Връщаме се в терминала, за да можем да работим натискайки **q**

### 3.1 Настройване на NGINX

За да може NGINX да работи по желания от нас начин е необходимо да променим част от неговите настройки. Първо намираме default файла, който трябва да се намира в следната директория: 

```
cd /etc/nginx/sites-available
```

След като намерим файла го отваряме с едитора:

```
sudo nano default
```

В този файл са необходими няколко промени. При първоначалното отваряне на файла в едитора, той ще изглежда по следния начин: 

![Image#3](/src/img/IMG_003.png)  

След нашите промени, той трябва да бъде така: 

![Image#4](/src/img/IMG_004.png) 

От тук може да копирате промените: 

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
      alias /srv/www/iD/;
}
```
След като запазим файла е необходимо да рестартираме NGINX, за да се отразят промените.

```
sudo systemctl restart nginx
```

## 4. Клониране на проекта Aquagis-Backend на сървъра

Следващата стъпка в нашата инсталация е да качим проекта Aquagis-Backend на сървъра. 

Репозиторито за проекта, може да бъде достъпено на следния [адрес](https://github.com/aqua-gis/aquagis-backend) 

Този проект трябва да бъде клониран в следната директория: 

```
cd /srv/apps/
```
```
sudo git clone https://github.com/aqua-gis/aquagis-backend.git
```

## 5. Инсталиране на PostgreSQL на сървъра

За да продължим с инсталацията на приложението е необходимо да инсталираме PostgreSQL на нашия сървър. За целта е необходимо да инсталираме PostgreSQL v13. В зависимост от операционната система на сървъра и нейната версия , можем да проверим как се инсталира онлайн или да следваме стъпките, показани в [този линк](https://computingforgeeks.com/how-to-install-postgresql-13-on-ubuntu/) и описани по-долу, ако системата е същата. 


### 5.1. Добавяне PostgreSQL 13 репозитори на Ubuntu 20.04

```
sudo apt -y install vim bash-completion wget

wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
```

### 5.2 Инсталиране на PostgreSQL 13 на Ubuntu 20.04

```
sudo apt update

sudo apt install postgresql-13 postgresql-client-13
```

След като инсталацията завърши е необходимо да променим паролата на автоматично създания акаунт по подразбиране. За тази цел е необходимо да се сменим акаунта си в терминала с този на postgres, използвайки следната команда: 

```
sudo su - postgres
```

Генерираме си random парола в браузъра и я използваме, като нова за акаунта по подразбиране: 

```
psql -c "alter user postgres with password 'Paste your password here!'"
```

След като променим основната парола, продължаваме създавайки нов потребител за базата. За целите на проекта **gis**, като потребителско име и парола за акаунта.


```
createuser --interactive --pwprompt
```

Създаваме потребителя, като въвеждаме потребителското име, последвано от паролата два пъти и даваме "superuser" парава на новосъздадения акаунт. 

След като потребителя е създаден е необходимо да променим няколко настройки в два файла, за да може да се вържем към базата. Файловете, които имат нужда от настройване могат да бъдат открити в следната директория: 

```
 cd /etc/postgresql/13/main
```

В тази директория ще работим съответно с **pg_hba.conf** и **postgresql.conf**.

```
 sudo nano pg_hba.conf
```

В първия файл трябва да добавим следния ред и да променим настройките, така че всички потребители да могат да се свържат към нея локално без парола:

```
host    all             all             10.0.0.0/0              md5
```

![Image#5](/src/img/IMG_005.png)  

След това продължаваме с промените във втория файл, където е необходимо да променим опцията на listen_addresses, както следва: 

![Image#6](/src/img/IMG_006.png) 
![Image#7](/src/img/IMG_007.png)  

```
listen_addresses = '*'                  # what IP address(es) to listen on;
```

След като направим и запазим промените, можем да се опитаме да се вържем към базата използвайки pgAdmim, Dbeaver или др. 

## 6.Създаване на базата и инсталиране на postgis
* Екранните снимки са взети от DBeaver, програмата която използваме за връзка с баата по време на ръководството.

### 6.1 Създаване на базата данни

След като успешно се вържем към създаденото в предните стъпки Postgres е необходимо да създадем нова база данни, както следва: 

![Image#8](/src/img/IMG_008.png)  

### 6.2 Конфигуриране на database.yml 

След като създадем новата база данни е необходимо да променим настройките в database.yml да сочат към нея:

```
sudo nano config/database.yml
```

![Image#12](/src/img/IMG_012.png) -> ![Image#13](/src/img/IMG_013.png)  

### 6.3 Инсталиране на Postgis

След като базатас е създадена и настроена е необходимо да инсталираме Postgis, както следва: 

За PostgreSQL 13:

```
sudo apt update
sudo apt install postgis postgresql-13-postgis-3
```

### 6.4 Добавяне на разширения в базата 

След инсталацията на Postgis трябва да добавим няколко разширения към създадената база данни.

Списък на разширения: 
- postgis
- btree_gist
- hstore

![Image#9](/src/img/IMG_009.png)  
![Image#10](/src/img/IMG_010.png)  
![Image#11](/src/img/IMG_011.png)  

### 6.5 Execute functions.sql

```
cd /srv/apps/aquagis-backend/db/functions
psql -U gis -d aquagis_test_dev -f functions.sql
```

## 7. Инсталиране на Vagrant

!!! За следващата част от инсталирането е възможно да се използва готов пакет, чрез vagrant който автоматично инсталира необходимите неща. Тук има възможност Vagrant да не работи на дадения ни сървър и да трябва тези елементи да бъдат инсталирани ръчно. (Описани по-долу в точка **7v2.**)

За да инсталираме vagrant е необходимо да изпълним следните команди:

```
sudo apt install vagrant
```

```
sudo apt install qemu qemu-kvm libvirt-clients libvirt-daemon-system virtinst bridge-utils

sudo systemctl enable libvirtd
sudo systemctl start libvirtds
```
```
sudo vagrant up
```

Ако инсталацията завърши успешно и успеем да стартираме vagrant, можем да изпълним тази команда, за да започне автоматичното инсталиране: 

vagrant provision

Ако тази операция завърши успешно, би трябвало базата ни да е заредена с всички необходими таблици и можем да продължим с останалите компоненти в точка **8**.

## 7v2. Инсталиране на компоненти без Vagrant

В случай, че инсталацията на vagrant и работата с него е невъзможна е необходимо да изпълним следните команди в терминала, за да инсталираме неонбходимите ни елемти: 

```
sudo apt-get install -y ruby2.7 libruby2.7 ruby2.7-dev \
                     libmagickwand-dev libxml2-dev libxslt1-dev nodejs yarnpkg \
                     apache2 apache2-dev build-essential git-core firefox-geckodriver \
                     libsasl2-dev imagemagick libffi-dev libgd-dev libarchive-dev libbz2-dev libpq-dev postgresql postgresql-contrib
```
```
sudo gem2.7 install rake
```
```
sudo gem2.7 install --version "~> 2.1.4" bundler
```
```
sudo bundle install --retry=10 --jobs=2
```
```
bundle exec rake yarn:install
```

Следващата стъпка е да променим отново настройките в в database.yml, тъй като в точка **6.2** го направихме, така че да работи използвайки vagrant. 

```
sudo nano config/database.yml
```

![Image#14](/src/img/IMG_014.png) -> ![Image#15](/src/img/IMG_015.png)  

След тази промяна можем да стартираме мигрирането и създаването на таблиците в базата данни: 

```
bundle exec rake db:migrate
```

Уверяваме се че базата ни е заредена с таблции и след това можем да стартираме приложението на OpenStreetMap, изпълнявайки следната команда: 

```
nohup bundle exec rails server -b 0.0.0.0 &
```

След като стартираме OpenStreetmap, влизаме през браъзъра в приложение достъпвайки го на адрес IP на сървъра, последвано от порт 3000 на който го настроихме по-рано: 

![Image#16](/src/img/IMG_016.png)  
