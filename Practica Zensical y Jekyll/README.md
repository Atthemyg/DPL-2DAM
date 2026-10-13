## Fase 1. Despliegue de la documentación con Zensical

### Obtener un site con Zensical

Primero nos situamos en la carpeta del proyecto donde tendremos nuestros archivos Markdown en un directorio `docs/` que usará Zensical para crear el site. A continuación, ejecutaremos `uv run zensical build` para generar el site en una carpeta `site/` en la raiz del proyecto.

Luego, utilizaremos el comando `uv run zensical serve` con el que podremos obtener una preview del site en nuestro `localhost`, por defecto, en el puerto `8000`.