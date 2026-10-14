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