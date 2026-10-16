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

## 3. Previsualizar el sitio en local

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