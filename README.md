## README - Crearea raportului

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
Se creează un director `containers04` și se clonează pe computer.

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
- Pornirea Apache2 și MariaDB automat la inițializare.

### **5. Crearea Dockerfile**
Adăugări:
- Instalarea supervisor.
- Montarea volumelor `mysql` și `logs`.
- Copierea fișierelor de configurare și a scriptului de pornire.
- Copierea și extragerea WordPress.
- Crearea directorului `/var/run/mysqld` cu permisiuni pentru `mysql:mysql`.
- Expunerea portului 80.
- Pornirea containerului cu `supervisord`.

### **6. Crearea bazei de date WordPress**
Conectarea la container și executarea comenzilor MySQL:
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
- Se completează detaliile bazei de date.
- Se salvează fișierul `wp-config.php` în `files/wp-config.php`.
- Se adaugă fișierul în Dockerfile:
  ```sh
  COPY files/wp-config.php /var/www/html/wordpress/wp-config.php
  ```

### **8. Testarea finală**
- Se recreează imaginea containerului.
- Se pornește containerul.
- Se verifică funcșionarea site-ului WordPress.

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








