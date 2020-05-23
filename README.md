<p align="center"><img src="https://enmemolab.com/app/github/starter.png"></p>

# Montaje-Droplet
Montaje de el droplet en DigitalOcean con Ubuntu 18.04, Nginx, php 7.3 y cerbot  para Laravel + Vue + y Pre-configuración de Websocket.

 [Si no tienes una cuenta en DigitalOcean](https://m.do.co/c/1c13fe7e733c)

 - Este link te dara 2 meses gratis de uso.
 
 ## Tutorial para crear un servidor o droplet
 [Link de youtube con el videotutorial](https://youtu.be/Kh7ujZiaOk8)
 
## Una vez creado el droplet
1. Cambia los DNS del dominio para que apunte a tu droplet
2. Ingresa a la consola que manejas y escribe ssh root@**IP asignada**
3. Una vez dentro del terminal siempre es aconsejable actualizar la instalación de Ubuntu, para esto hacemos:
```
sudo apt update
sudo apt upgrade
```

## Instalamos Nginx
```
sudo apt install nginx -y
```
- Comandos para poder iniciar, parar, o poner enable Nginx:
```
sudo systemctl stop nginx.service
sudo systemctl start nginx.service
sudo systemctl restart nginx.service
sudo systemctl reload nginx.service
sudo systemctl disable nginx.service
sudo systemctl enable nginx.service
```

## Abrimos los puertos que necesitamos
```
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw allow OpenSSH
sudo ufw allow 3306
sudo ufw allow 6001
```
**Importante:** debemos ir al panel de control del droplet y en networking/firewalls y agregamos dos reglas para poder abrir los puertos para accesar a mysql y para las comunicaciones del websocket

| Nombre  | Tipo | Puerto | IPv4 | IPv6 |
| ------------- | ------------- | ------------- | ------------- |------------- |
| MySQL  | TCP  | 3306  | All IPv4  |All IPv6  |
| Websocket  | TCP  | 6001  | All IPv4  |3306  |

- **Importante:** Habilitamos nuestra IP de nuestra casa para poder acceder a la mysql desde el programa que usemos para manejo de base de datos.
`sudo ufw allow from **IP que usas en tu casa** to any port 3306`

###### Habilitamos el UFW de Nginx
`sudo ufw enable`
###### Confirmamos el UFW de Nginx
`sudo ufw status`

## Instalamos PHP 7.3
```
sudo add-apt-repository ppa:ondrej/php
sudo apt install software-properties-common
sudo apt update
sudo apt install -y php7.3 php7.3-cli php7.3-common php7.3-curl php7.3-mysql php7.3-odbc php7.3-pgsql php7.3-sybase php7.3-mbstring php7.3-fpm php7.3-xml php7.3-xmlrpc php7.3-xsl php7.3-zip php7.3-opcache php7.3-tidy php7.3-bcmath php7.3-gd php7.3-soap php7.3-imap php7.3-intl php7.3-recode php7.3-bz2 php7.3-json php7.3-readline php7.3-imagick php7.3-dom php7.3-simplexml php7.3-ssh2 php7.3-xmlreader php7.3-exif php7.3-ftp php7.3-iconv php7.3-posix php7.3-sockets php7.3-tokenizer php7.3-dev 
```
```
sudo apt-get install python
sudo apt-get install libglib2.0-dev
```
## Editamos el PHP FPM
`sudo nano /etc/php/7.3/fpm/php.ini`

```
upload_max_filesize = 32M 
post_max_size = 48M 
memory_limit = 256M 
max_execution_time = 600 
max_input_vars = 3000 
max_input_time = 1000
```
**Reiniciamos!**
```
sudo php-fpm7.3 -t 
sudo service php7.3-fpm restart
```

## Instalamos MYSQL

```
sudo apt-get -y install mysql-server-5.7
sudo mysql
SELECT user,authentication_string,plugin,host FROM mysql.user;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'CONTRASEÑA';
FLUSH PRIVILEGES;
SELECT user,authentication_string,plugin,host FROM mysql.user;
exit
```
- Entramos como administrador y creamos un segundo usuario:
```
mysql -u root -p
CREATE USER 'USUARIO'@'localhost' IDENTIFIED BY 'CONTRASEÑA';
GRANT ALL PRIVILEGES ON *.* TO 'USUARIO'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit
```

## Instalamos un nuevo usuario
```
adduser username
usermod -aG sudo username
```

## Instalamos complementos que usaremos más adelante
```
sudo apt update
sudo apt install redis-server -y
sudo apt install supervisor -y
```

## Instalamos COMPOSER
```
sudo mkdir -p /var/www/descargas
cd /var/www/descargas
sudo apt install composer -y
```

## Instalamos Node
```
curl -sL https://deb.nodesource.com/setup_13.x | sudo -E bash -
sudo apt-get install -y nodejs
```
- Verificamos la instalación:
```
node -v
npm -v
rm -r /var/www/descargas
```

## Generamos una clave para poder operar con GIT 
`ssh-keygen -t rsa`
- Podemos acceder a las claves con:
`cat /root/.ssh/id_rsa.pub`
`cat /root/.ssh/id_rsa`

## Creamos los usuarios que podrán operar con git
```
git config --global user.email CORREO ELECTRONICO
git config --global user.name USUARIO
```

**Nota:** Si usas github debes pegar la clave pública en su plataforma, así podrás conectarte y enviar los push. 

## Instalamos Cerbot
```
sudo add-apt-repository ppa:certbot/certbot
sudo apt update
sudo apt install python-certbot-nginx
```

## Habilitamos nuestro sitio en Nginx
`sudo nano /etc/nginx/sites-available/DOMINIO`
- Provisionalmente ponemos estas líneas mientras creamos los certificados.
```
server {

        root /var/www/DOMINIO;
        index index.html index.htm index.nginx-debian.html;

        server_name DOMINIO.com www.DOMINIO.com;

        location / {
                try_files $uri $uri/ =404;
        }
}
```
`sudo ln -s /etc/nginx/sites-available/DOMINIO /etc/nginx/sites-enabled/`
- Quitamos el link al default
`unlink /etc/nginx/sites-enabled/default`
- Editamos el nginx.conf
`sudo nano /etc/nginx/nginx.conf`
- Buscamos  **server_names_hash_bucket_size** y quitar el #
**Reiniciamos!**
```
sudo nginx -t
sudo systemctl restart nginx
```

## Actualizamos el servidor
```
sudo apt update
sudo apt upgrade
```

## Instalamos nuestro Laravel
`cd /var/www`
- Yo casi siempre borro lo que no necesito y trato de mantener mi servidor bien organizado, asi que yo borro siempre esta carpeta:
`rm -r html`
- Instalación de Laravel
```
composer create-project --prefer-dist laravel/laravel DOMINIO
cd /var/www/DOMINIO

cp .env.example .env
```

- Instalación del Frontend Scaffolding
```
composer require laravel/ui --dev
php artisan ui vue
```

- Normalmente nuestro lío siempre son los permisos de carpetas, estos son los que yo uso y por lo general casi todos hacemos procesos diferentes, estos pueden cambiar.

- Dar permiso al usuario, y pegar el codigo justo debajo de root
```
sudo adduser USER www-data
```
```
sudo chgrp www-data /var
sudo chgrp -R www-data /var/www
sudo chgrp -R www-data /var/www/DOMINIO
sudo chmod 775 /var
sudo chmod 775 /var/www
sudo chmod 775 /var/www/DOMINIO

sudo chown -R www-data:www-data /var/www/DOMINIO
sudo find /var/www/DOMINIO/ -type d -exec chmod 755 {} \;
sudo find /var/www/DOMINIO/ -type f -exec chmod 644 {} \;
sudo chgrp -R www-data /var/www/DOMINIO/storage /var/www/DOMINIO/bootstrap/cache
sudo chmod -R ug+rwx /var/www/DOMINIO/storage /var/www/DOMINIO/bootstrap/cache
sudo usermod -a -G www-data root
sudo chown -R root:www-data /var/www/DOMINIO
sudo find /var/www/DOMINIO/ -type f -exec chmod 664 {} \;    
sudo find /var/www/DOMINIO/ -type d -exec chmod 775 {} \;
```

## Creamos nuestros certificados
`sudo certbot --nginx -d DOMINIO.com -d www.DOMINIO.com`
`sudo certbot --nginx -d sub.DOMINIO.com`

## Editamos nuestra configuración del sitio en Nginx
`sudo nano /etc/nginx/sites-available/DOMINIO`
- **OJO** debemos tener en cuenta donde quedaron los certificados para dejar las mismas rutas.
- Esta configuración tiene listo el manejo de websocket. Esto no afecta si no usas websocket, asi que si no quieres usar websocket y no quieres que este presente, puedes cambiar esta configuracion.

```
map $http_upgrade $type {
  default "web";
  websocket "ws";
}

server {

    listen 80;
    listen [::]:80;

    root /var/www/DOMINIO/public;
    index index.php index.html index.htm index.nginx-debian.html;

    charset utf-8;
    
    server_name DOMINIO.com www.DOMINIO.com;

    location ~ \.php$ {
        try_files $uri /index.php =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php7.3-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    
    location / {
        try_files /nonexistent @$type;
    }
  
    location @web {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location @ws {
        proxy_pass             http://127.0.0.1:6001;
        proxy_set_header Host  $host;
        proxy_read_timeout     60;
        proxy_connect_timeout  60;
        proxy_redirect         off;

        # Allow the use of websockets
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/DOMINIO/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/DOMINIO/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
    
    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    # Redirect non-https traffic to https
    # if ($scheme != "https") {
    #     return 301 https://$host$request_uri;
    # } # managed by Certbot
}
```
- **OJO** debemos reiniciar Nginx
`sudo nginx -s reload`

## Iniciamos nuestro repositorio y hacemos nuestro primer push
1.  Vamos a GitHub y creamos un repositorio VACIO.
2.  Ejecutamos los siguientes comandos:
```
cd /var/www/DOMINIO
git init
git add .
git commit -m "Initial commit"
git remote add origin COPIAMOS LA RUTA DEL REPOSITORIO
git remote -v
git push -u origin master
```
- Comandos que podremos usar más adelante con git:
```
git fetch --all
git reset --hard origin/master
git pull
```

## Preconfiguracion del supervisor
```
service supervisor status
sudo groupadd supervisor_DOMINIO
nano /etc/supervisor/supervisord.conf
```
- Cambiar :
`chmod=0770`
- Reiniciamos :
`sudo service supervisor restart`

#### Y eso es todo por el momento, el siguente instaleremos websocket y lo dejaremos funcionando.
