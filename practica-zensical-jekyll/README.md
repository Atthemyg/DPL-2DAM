## Fase 1. Despliegue de la documentación con Zensical

### 1. Preparar el proyecto

Primero, nos situamos en la carpeta del proyecto donde tendremos nuestros archivos Markdown de la documentación. Esta vive en el directorio `docs/`, que es el que Zensical usa como origen para crear el sitio.


### 2. Obtener un site con Zensical

Para construir el sitio web estático (HTML, CSS y JavaScript) a partir de los Markdown de `docs/`, ejecutamos el build de Zensical con `uv`.

```bash
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

```bash
ls site
```

**Resultado:** 

```
404.html  get-site    markdown     sitemap.xml  trabajando-con-git
assets    index.html  search.json  tema-visual
```

### 3. Previsualizar el sitio en local

Debemos comprobar que la documentación se ve correctamente antes de publicarla. Para ello, lanzaremos el servidor de desarrollo de Zensical:

```bash
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

```bash
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

```bash
sudo apt update
sudo apt install -y nginx
nginx -v
```

Arrancamos el servicio y comprobar que responde:

```bash
sudo systemctl enable --now nginx
sudo systemctl status nginx --no-pager
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

```bash
sudo mkdir -p /var/www/docs
sudo cp -r site/. /var/www/docs/
sudo chown -R www-data:www-data /var/www/docs
```

Y crearemos el fichero de configuración dentro del repositorio, en `nginx/docs.conf`:

```bash
mkdir -p nginx
nano nginx/docs.conf
```

Dentro de `docs.conf` pegamos:

```
server {
    listen 80;
    server_name localhost;

    root /var/www/docs;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Hecho esto, lo instalaremos en NGINX con un enlace simbólico, y desactivaremos el sitio por defecto para que no interfiera:

```bash
sudo cp nginx/docs.conf /etc/nginx/sites-available/docs.conf
sudo ln -s /etc/nginx/sites-available/docs.conf /etc/nginx/sites-enabled/docs.conf
sudo rm -f /etc/nginx/sites-enabled/default
```

Validamos y recargamos:

```bash
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

```bash
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

```bash
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

```bash
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

### 3. Generar el sitio del blog

Para construir el sitio estático del blog, debemos descargar las dependencias necesarias:

```bash
bundle install
bundle config set --local path vendor/bundle   # configura Bundler para instalar las gemas dentro del proyecto
bundle install
```

A continuación, creamos el sitio:

```bash
cd blog
bundle exec jekyll build
ls _site
cd ..
```

Jekyll deja el resultado en `blog/_site/`.

### 4. Publicar el blog en el mismo NGINX

Añadiremos un segundo `server` al mismo NGINX, de modo que cada sitio se sirva de forma independiente según el nombre de host (`localhost` para la documentación y `blog.local` para el blog).

Primero, copiaremos el sitio generado y ajustaremos permisos:

```bash
sudo mkdir -p /var/www/blog
sudo cp -r blog/_site/. /var/www/blog/
sudo chown -R www-data:www-data /var/www/blog
```

Luego, crearemos `nginx/blog.conf` en el repositorio:

```bash
cd nginx  # ya está creada
nano nginx/blog.conf
```

Dentro de `blog.conf` pegamos:

```
server {
    listen 80;
    server_name blog.local;

    root /var/www/blog;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Por último, instalamos, validamos y recargamos:

```bash
sudo cp nginx/blog.conf /etc/nginx/sites-available/blog.conf
sudo ln -s /etc/nginx/sites-available/blog.conf /etc/nginx/sites-enabled/blog.conf
sudo nginx -t
sudo systemctl reload nginx
```

**Resultado:**

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### 5. Comprobar que ambos sitios funcionan de forma independiente

Verificaremos que cada URL sirve su propio contenido en el mismo NGINX:

```bash
curl -s http://localhost/ | grep -m1 -i "<title>"
curl -s -H "Host: blog.local" http://localhost/ | grep -m1 -i "<title>"
```

Los dos `<title>` deben ser distintos: el de la documentación y el del blog. Para abrir el blog en el navegador, añadiremos `127.0.0.1 blog.local` a `/etc/hosts` y visitaremos `http://blog.local`; la documentación sigue en `http://localhost`.

**Resultado:**

```text
<title>Get started - Documentation</title>
<title>Mi primera entrada | Mi blog personal sobre desarrollo web.</title>
```


---

## Actualizar o eliminar contenido de la documentación

Cuando se quita una página de la documentación no basta con borrar el fichero Markdown. El contenido vive en tres sitios a la vez, y cada uno hay que actualizarlo:

| Dónde | Qué contiene | Ruta |
|-------|--------------|------|
| Repositorio | Los Markdown originales | `docs/` |
| Sitio generado | El HTML que crea Zensical | `site/` |
| NGINX | La copia que ve el navegador | `/var/www/docs/` |

Si solo se borra el Markdown, la web no cambia: `site/` y `/var/www/docs/` conservan las páginas antiguas. Por eso el proceso se hace en tres pasos.

### 1. Quitar la página del menú

Se comprueba si la página aparece en el menú, que está en la sección `nav` del fichero de configuración de Zensical (`zensical.toml`). Si aparece, hay que eliminar también su línea.

```toml
nav = [
  { "Get started" = "index.md" },
  # { "Tema Visual" = "tema-visual.md" },   <- línea eliminada
]
```

El menú es una lista de ficheros que Zensical espera encontrar. Si una entrada apunta a un Markdown que ya no existe, el build muestra avisos y puede generar enlaces rotos en la navegación. Lo mismo ocurre con los enlaces internos de otras páginas hacia la página borrada (por ejemplo, un índice con anclas a secciones que ya no existen), que también aparecen como avisos durante el build.

### 2. Regenerar el sitio desde cero

```bash
rm -rf site
uv run zensical build
```

`zensical build` escribe en `site/`, pero no borra lo que ya hay. Si se omite `rm -rf site`, las páginas eliminadas siguen en esa carpeta y acaban publicándose igualmente. Partir de una carpeta vacía garantiza que `site/` refleja exactamente el contenido actual de `docs/`.

### 3. Vaciar y volver a publicar en NGINX

```bash
sudo rm -rf /var/www/docs/*
sudo cp -r site/. /var/www/docs/
sudo chown -R www-data:www-data /var/www/docs
```

- `rm -rf /var/www/docs/*` vacía la carpeta que sirve NGINX, porque `cp -r` solo añade y sobrescribe ficheros, nunca elimina los que ya no existen. Sin este paso, las páginas borradas seguirían accesibles desde el navegador.
- `cp -r site/. /var/www/docs/` copia el sitio nuevo. La barra y el punto (`/.`) copian también los ficheros ocultos y evitan crear una subcarpeta `site` dentro del destino.
- `chown -R www-data:www-data` devuelve la propiedad al usuario con el que se ejecuta NGINX. Los ficheros copiados con `sudo` pertenecen a `root`, y sin este paso NGINX podría responder con un error 403.

<br>

**Bloque de comandos completos:**

```bash
rm -rf site
uv run zensical build
sudo rm -rf /var/www/docs/*
sudo cp -r site/. /var/www/docs/
sudo chown -R www-data:www-data /var/www/docs
```

### 4. Comprobación

```bash
ls /var/www/docs
curl -I http://localhost
```

**Resultado:**

```text
HTTP/1.1 200 OK
Server: nginx
Date: Fri, 16 Oct 2026 22:23:43 GMT
Content-Type: text/html
Content-Length: 49210
Last-Modified: Fri, 16 Oct 2026 22:16:30 GMT
Connection: keep-alive
ETag: "6ad2a23e-c03a"
Accept-Ranges: bytes
```

Para confirmar que una página eliminada ya no existe, se pide su dirección y debe devolver un 404:

```bash
curl -I http://localhost/tema-visual/
```

**Resultado:**

```text
HTTP/1.1 404 Not Found
Server: nginx
Date: Fri, 16 Oct 2026 22:24:16 GMT
Content-Type: text/html
Content-Length: 146
Connection: keep-alive
```

Si el navegador sigue mostrando la versión anterior, se recarga sin caché con `Ctrl+Shift+R`, porque el tema de Zensical usa navegación instantánea y puede conservar páginas ya visitadas.
