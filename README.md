## **Numele lucrării de laborator**
**Pregătirea unui container pentru rularea unui site web bazat pe Apache HTTP Server + PHP (mod_php) + MariaDB.**

## **Scopul lucrării**
Scopul acestei lucrări este de a pregăti un container Docker capabil să ruleze un site web bazat pe Apache HTTP Server, PHP (mod_php) și MariaDB, cu stocarea bazei de date MariaDB pe un volum montat. Serverul trebuie să fie disponibil pe portul 8000.

## **Sarcina**
1. Crearea unui fișier Dockerfile pentru construirea imaginii containerului.
2. Instalarea și configurarea Apache2, PHP, MariaDB.
3. Instalarea și verificarea funcșionării site-ului WordPress.

## **Descrierea executării lucrării**

### **1. Crearea unui depozit de cod sursă**
Se creează un director `containers04` cu fișierul `Dockerfile`:
```sh
# create from debian image
FROM debian:latest

# install apache2, php, mod_php for apache2, php-mysql and mariadb
RUN apt-get update && \
    apt-get install -y apache2 php libapache2-mod-php php-mysql mariadb-server && \
    apt-get clean
```

Se construieste imaginea: `apache2-php-mariadb`:
```sh
docker build -t apache2-php-mariadb .
```
Se creaza un container apache2-php-mariadb din imaginea apache2-php-mariadb și se porneste în modul de fundal cu comanda bash:
```sh
docker run -d -p 80:80 --name apache2-php-mariadb apache2-php-mariadb
```


### **2. Extragerea fișierelor de configurare**
Se creează structura de directoare pentru fișierele de configurare:
```sh
containers04/
  files/
    apache2/
    php/
    mariadb/
```
Se execută comenzile pentru copierea fișierelor de configurare:
```sh
docker cp apache2-php-mariadb:/etc/apache2/sites-available/000-default.conf files/apache2/
docker cp apache2-php-mariadb:/etc/apache2/apache2.conf files/apache2/
docker cp apache2-php-mariadb:/etc/php/8.2/apache2/php.ini files/php/
docker cp apache2-php-mariadb:/etc/mysql/mariadb.conf.d/50-server.cnf files/mariadb/
```

Apoi am oprit și șters containerul apache2-php-mariadb:
```sh
docker stop apache2-php-mariadb
docker rm apache2-php-mariadb
```
### **3. Modificări ale fișierelor de configurare**

#### **Apache2**
- Modificări aduse fișierului `000-default.conf`:
  ```sh
  ServerName localhost
  DirectoryIndex index.php index.html
  ```
- Adăugare la sfârșitul fișierului `apache2.conf`:
  ```sh
  ServerName localhost
  ```

#### **PHP**
- Modificări aduse fișierului `php.ini`:
  ```sh
  error_log = /var/log/php_errors.log
  memory_limit = 128M
  upload_max_filesize = 128M
  post_max_size = 128M
  max_execution_time = 120
  ```

#### **MariaDB**
- Eliminarea `#` din fața liniei `log_error = /var/log/mysql/error.log`.

### **4. Crearea scriptului de pornire**
Se creează directorul `files/supervisor/` și fișierul `supervisord.conf` cu următoarele setări:
```sh
[supervisord]
nodaemon=true
logfile=/dev/null
user=root

# apache2
[program:apache2]
command=/usr/sbin/apache2ctl -D FOREGROUND
autostart=true
autorestart=true
startretries=3
stderr_logfile=/proc/self/fd/2
user=root

# mariadb
[program:mariadb]
command=/usr/sbin/mariadbd --user=mysql
autostart=true
autorestart=true
startretries=3
stderr_logfile=/proc/self/fd/2
user=mysql
```

### **5. Crearea Dockerfile**
Adăugări:
- Instalarea supervisor.
- Montarea volumelor `mysql` și `logs`:
```sh
# mount volume for mysql data
VOLUME /var/lib/mysql

# mount volume for logs
VOLUME /var/log
```
- Copierea fișierelor de configurare și a scriptului de pornire.
- Copierea și extragerea WordPress:
```sh
# add wordpress files to /var/www/html
ADD https://wordpress.org/latest.tar.gz /var/www/html/
RUN tar -xzvf /var/www/html/latest.tar.gz -C /var/www/html/ 
```
-
- După copierea fișierelor WordPress adăugați copierea fișierelor de configurare apache2, php, mariadb, așa cum și a scriptului de pornire:
```sh
# copy the configuration file for apache2 from files/ directory
COPY files/apache2/000-default.conf /etc/apache2/sites-available/000-default.conf
COPY files/apache2/apache2.conf /etc/apache2/apache2.conf

# copy the configuration file for php from files/ directory
COPY files/php/php.ini /etc/php/8.2/apache2/php.ini

# copy the configuration file for mysql from files/ directory
COPY files/mariadb/50-server.cnf /etc/mysql/mariadb.conf.d/50-server.cnf

# copy the supervisor configuration file
COPY files/supervisor/supervisord.conf /etc/supervisor/supervisord.conf
pentru functionarea mariadb creati directorul /var/run/mysqld si setati permisiile pe el:
# create mysql socket directory
RUN mkdir /var/run/mysqld && chown mysql:mysql /var/run/mysqld
```
- Deschideti portul 80: `EXPOSE 80`;

- Se adauga comanda de pornire supervisord:
```sh
# start supervisor
CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```

### **6. Crearea bazei de date WordPress**
Conectarea la container și executarea comenzilor MySQL:
```sh
docker exec -it apache2-php-mariadb bash   
```
```sh
mysql -u root -p
```
```sh
mysql
CREATE DATABASE wordpress;
CREATE USER 'wordpress'@'localhost' IDENTIFIED BY 'wordpress';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### **7. Configurarea WordPress**
- Se accesează site-ul la `http://localhost/`.
- Se completează detaliile bazei de date:
```sh
Numele bazei de date: wordpress;
Utilizatorul bazei de date: wordpress;
Parola bazei de date: wordpress;
Adresa bazei de date: localhost;
Prefixul tabelelor: wp_.
```
- Se salvează fișierul `wp-config.php` în `files/wp-config.php`
- Se adaugă fișierul în Dockerfile:
  ```sh
  COPY files/wp-config.php /var/www/html/wordpress/wp-config.php
  ```

### **8. Testarea finală**
- Se recreează imaginea containerului.
- Se pornește containerul.
- Se verifică functionarea site-ului WordPress.
 
![Descrierea imaginii](CONTAINERS05/images/CAPTURE.JPG)
![Descrierea imaginii](images/CAPTURE1.JPG)
![Descrierea imaginii](images/CAPTURE2.JPG)

## **Răspunsuri la întrebări**

### **Ce fișiere de configurare au fost modificate?**
- `000-default.conf`
- `apache2.conf`
- `php.ini`
- `50-server.cnf`
- `wp-config.php`

### **Pentru ce este responsabilă instrucția DirectoryIndex din fișierul de configurare Apache2?**
Instrucția `DirectoryIndex` specifică fișierul implicit care trebuie servit când un director este accesat. În acest caz, `index.php` are prioritate față de `index.html`.

### **De ce este necesar fișierul wp-config.php?**
Acest fișier conține setările de configurare pentru WordPress, inclusiv detaliile de conectare la baza de date.

### **Pentru ce este responsabil parametrul post_max_size din fișierul de configurare PHP?**
`post_max_size` definește dimensiunea maximă a datelor care pot fi transmise printr-o cerere POST.

### **Deficiențele imaginii containerului creat**
- Imaginea este mare, deoarece conține Apache, PHP și MariaDB.
- Nu include un mecanism de auto-actualizare pentru WordPress.
- Nu are protecție suplimentară pentru MariaDB (ex. setări de securitate implicite).

## **Concluzii**
Această lucrare a demonstrat procesul de creare a unui container Docker capabil să ruleze un site WordPress cu Apache HTTP Server, PHP și MariaDB. Studentul a acumulat experiență în crearea, configurarea și gestionarea unui mediu de hosting pentru WordPress folosind containere Docker.








