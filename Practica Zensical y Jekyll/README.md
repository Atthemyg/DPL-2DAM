## Fase 1. Despliegue de la documentación con Zensical

### 1. Preparar el proyecto

Primero, nos situamos en la carpeta del proyecto donde tendremos nuestros archivos Markdown de la documentación. Esta vive en el directorio `docs/`, que es el que Zensical usa como origen para crear el sitio.


### 2. Obtener un site con Zensical

Para construir el sitio web estático (HTML, CSS y JavaScript) a partir de los Markdown de `docs/`, ejecutamos el build de Zensical con `uv`.

```
uv run zensical build
``` 
Esto genera la carpeta `site/` en la raíz del proyecto, que es el contenido que más adelante servirá NGINX.

**Resultado:** 

```
Using CPython 3.14.7
Creating virtual environment at: .venv
Installed 11 packages in 495ms
Build started
No issues found
Build finished in 0.49s
```

Comprobamos que se creó el `site/`

```
ls site
```

**Resultado:** 

```
404.html  get-site    markdown     sitemap.xml  trabajando-con-git
assets    index.html  search.json  tema-visual
```

### 3. Previsualizar el sitio en local

Debemos comprobar que la documentación se ve correctamente antes de publicarla. Para ello, lanzaremos el servidor de desarrollo de Zensical:

```
uv run zensical serve
```

Por defecto la previsualización queda disponible en `http://localhost:8000`.

**Resultado:**

```
Serving /home/alumno/2DAM-REPOS/DPL-2DAM/Practica Zensical y Jekyll/gitanddocs/site on http://localhost:8000
Build started
No issues found
```

Podemos hacer una verificación desde otra terminal con:

```
curl -I http://localhost:8000
```

**Resultado:**

```
HTTP/1.1 200 OK
Content-Length: 1303
Content-Type: text/html; charset=utf-8
Date: Fri, 16 Oct 2026 10:38:37 GMT
Last-Modified: Fri, 16 Oct 2026 10:37:17 GMT
```

### 4. Instalar NGINX y verificar el servicio

Ahora instalaremos NGINX en el entorno donde se desplegará la documentación:

```
sudo apt update
sudo apt install -y nginx
nginx -v
```

Arrancamos el servicio y comprobar que responde:

```
systemctl enable --now nginx
systemctl status nginx --no-pager
```

**Resultado:**

```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-10-16 11:49:00 WEST; 55s left
 Invocation: e75903278a9a4f2db68f5704dab9ff62
       Docs: man:nginx(8)
   Main PID: 1148 (nginx)
      Tasks: 5 (limit: 7325)
     Memory: 6.4M (peak: 6.9M)
        CPU: 76ms
     CGroup: /system.slice/nginx.service
             ├─1148 "nginx: master process /usr/sbin/nginx -g daemon on; master…
             ├─1149 "nginx: worker process"
             ├─1150 "nginx: worker process"
             ├─1151 "nginx: worker process"
             └─1152 "nginx: worker process"

```

**Resultado** desde otra terminal con `curl -I http://localhost`:

```
HTTP/1.1 200 OK
Server: nginx
Date: Fri, 16 Oct 2026 10:49:43 GMT
Content-Type: text/html
Content-Length: 33157
Last-Modified: Tue, 13 Oct 2026 10:26:47 GMT
Connection: keep-alive
ETag: "6ace0767-8185"
Accept-Ranges: bytes
```

### 5. Publicar `site/` con NGINX

A continuación, copiaremos el sitio generado a una ruta propia de NGINX y crear un `server` que lo sirva.

Para hacerlo tenemos que copiar el contenido de `site/` a la ruta que servirá NGINX y ajustar permisos:

```
sudo mkdir -p /var/www/docs
sudo cp -r site/. /var/www/docs/
sudo chown -R www-data:www-data /var/www/docs
```

Y crearemos el fichero de configuración dentro del repositorio, en `nginx/docs.conf`:

```
mkdir -p nginx
nano nginx/docs.conf
```

Dentro de `docs.conf` pegamos:

```
server {
    listen 80;
    server_name docs.local;

    root /var/www/docs;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Hecho esto, lo instalaremos en NGINX con un enlace simbólico, y desactivaremos el sitio por defecto para que no interfiera:

```
sudo cp nginx/docs.conf /etc/nginx/sites-available/docs.conf
sudo ln -s /etc/nginx/sites-available/docs.conf /etc/nginx/sites-enabled/docs.conf
sudo rm -f /etc/nginx/sites-enabled/default
```

Validamos y recargamos:

```
sudo nginx -t
sudo systemctl reload nginx
```

**Resultado:**

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### 6. Comprobar el acceso a la documentación

Solo nos falta verificar que la documentación se sirve correctamente desde NGINX:

```
curl -I http://localhost
```
**Resultado:**

```
HTTP/1.1 200 OK
Server: nginx
Date: Fri, 16 Oct 2026 11:20:55 GMT
Content-Type: text/html
Content-Length: 33157
Last-Modified: Fri, 16 Oct 2026 11:00:41 GMT
Connection: keep-alive
ETag: "6ad203d9-8185"
Accept-Ranges: bytes
```

Con este último paso, ya podremos ver la documentación generada con Zensical y servida por NGINX al abrir `http://localhost` en nuestro navegador. 

---

## Fase 2. Despliegue de un blog personal con Jekyll

### 1. Instalar Jekyll

Debemos de instalar Ruby y Jekyll para poder construir el blog:

```
sudo apt install -y ruby-full build-essential zlib1g-dev
sudo gem install jekyll bundler
jekyll -v
```

**Resultado:**

```
jekyll 4.4.1
```

### 2. Crear el blog

Con Jekyll se puede generar un blog completo con un solo comando, que incluye la configuración, el tema por defecto, la página de inicio, la página "Acerca de" y una entrada de ejemplo:

```
jekyll new blog
cd blog
bundle install
```

A continuación, personalizaremos `blog/_config.yml` (título, descripción, autor):

```
title: Mi primer blog
email: alumno@example.com
description: >-
  Mi blog personal sobre desarrollo web.
author: Alumno
```

 y añadiremos al menos una entrada propia en `blog/_posts/` con el formato `AAAA-MM-DD-titulo.md`:

```
---
layout: post
title: "Mi primera entrada"
date: 2026-10-16
---

Contenido de la entrada.
```