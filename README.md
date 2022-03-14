# Ръководство за инсталиране на Open Street Map (OSM) и стартиране на iD Editor

## 1. Свързване към сървъра

За да започнем инсталацията на OSM, първо трябва да се вържем към сървъра използвайки терминално приложение (емулатор). В случая и за целите на това ръководство ще работим с PuTTY, но може да бъде използвано и друго приложение по ваш избор.

   [Линк за изтегляне на PuTTY](https://www.putty.org/)

След като инсталираме успешно емулатора, се връзваме към сървара използвайки предоставеното ни IP и данни за потребител и парола.


## 2. Проверка на вид и версия на операционна система

** !!!Тази стъпка е важна, защото инсталацията на част от компонентите се различава за различните операционни системи и версии **

За да проверим вида операционна система под която работи сървъра, изпълняваме следната команда:

```
cat /etc/os-release
```
Това ще ни покаже търсената от нас информация: 

 ![Image#1](/src/img/IMG_001.png)  
