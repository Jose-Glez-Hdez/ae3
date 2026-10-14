# AE3 — Galería con image_filter, HTTPS y goaccess

## 1. Instalación

```bash
apt-get update
apt-get install -y nginx curl openssl libnginx-mod-http-image-filter imagemagick goaccess
```

## 2. Generación de las imágenes

Las 20 imágenes de prueba (cuadradas, 400x400) se generan con ImageMagick:

```bash
sudo mkdir -p /var/www/images.alumno.local/img
for i in $(seq -w 1 20); do
  convert -size 400x400 gradient:"#2f6fed"-"#8bc34a" \
    -fill white -pointsize 64 -gravity center -annotate 0 "IMG $i" \
    /var/www/images.alumno.local/img/image$i.jpg
done
```

## 3. Publicación de los archivos web

Creamos dentro del directorio web los siguientes archivos: index.html, estilos.css y app.js, y los copiamos a la ruta /var/www/images.alumno.local/

```bash
cp web/{index.html,app.js,estilos.css} /var/www/images.alumno.local/
chmod -R a+rX /var/www/images.alumno.local
```

## 4. Generación de certificado autofirmado

Realizamos los siguientes comandos:

```bash
mkdir -p /etc/ssl/dpl
openssl req -x509 -nodes -newkey rsa:2048 -days 365 \
  -keyout /etc/ssl/dpl/images.alumno.local.key \
  -out /etc/ssl/dpl/images.alumno.local.crt \
  -subj "/CN=images.alumno.local" \
  -addext "subjectAltName=DNS:images.alumno.local,DNS:www.images.alumno.local"
chmod 400 /etc/ssl/dpl/images.alumno.local.key
cp /etc/ssl/dpl/images.alumno.local.crt ~/dpl/ae3/certs/
```

Con esto generamos el certificado y copiamos el certificado público en el directorio certs de nuestra carpeta raíz.

## 5. Crear el usuario de la galería

Con los siguientes comandos crearemos el usuario que nos pedirá cuando accedamos a la galería:

```bash
echo "alumno:$(openssl passwd -apr1 CAMBIAR_ESTO)" | tee /etc/nginx/htpasswd-images
chown root:www-data /etc/nginx/htpasswd-images
chmod 640 /etc/nginx/htpasswd-images
```

**RECUERDA CAMBIAR LA CONTRASEÑA**

## 6. Crear o modificar el archivo de configuración de Nginx

Modificaremos o crearemos el archivo /etc/nginx/sites-available/010-images.alumno.local con el siguiente contenido:

```
# Listas blancas: un valor no listado usa el default
map $arg_size $img_size {
    default 300;
    150     150;
    300     300;
    600     600;
}

map $arg_sharpen $img_sharpen {
    default 0;
    0       0;
    30      30;
    60      60;
    100     100;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name images.alumno.local;
    root /var/www/images.alumno.local;
    index index.html;

    ssl_certificate     /etc/ssl/dpl/images.alumno.local.crt;
    ssl_certificate_key /etc/ssl/dpl/images.alumno.local.key;

    location / {
        try_files $uri $uri/ =404;
    }

    # image_filter SOLO aquí
    location /img/ {
        auth_basic "Galeria de imagenes";
        auth_basic_user_file /etc/nginx/htpasswd-images;

        # Sin esto, image_filter convierte el 401 en un 415
        error_page 401 @sin_autorizacion;

        image_filter resize $img_size $img_size;
        image_filter_sharpen $img_sharpen;
        image_filter_buffer 5M;
    }

    location @sin_autorizacion {
        return 401;
    }

    access_log /var/log/nginx/images-alumno.access.log;
    error_log  /var/log/nginx/images-alumno.error.log;
}

# HTTP -> HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name images.alumno.local;
    return 301 https://$host$request_uri;
}

# www -> dominio base
server {
    listen 80;
    listen [::]:80;
    server_name www.images.alumno.local;
    return 301 https://images.alumno.local$request_uri;
}
```

Generamos el enlace simbólico en /etc/nginx/sites-enabled/, y reiniciamos el servicio de Nginx:

```bash
ln -s /etc/nginx/sites-available/010-images.alumno.local /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```