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