# Conectarse por SSH a una máquina Debian (desde cero)

Queremos entrar de forma remota en una máquina Debian desde otro equipo. Para eso hace falta que en esa máquina esté instalado y en marcha el servidor SSH (`sshd`). En esta guía lo hacemos desde cero sobre una máquina Debian que levantamos con Docker a modo de ejemplo.

Esta práctica también "vale para una VPS real". Los comandos dentro de la máquina son los mismos que en un servidor Debian real. Solo cambian dos cosas: cómo te conectas (aquí a `127.0.0.1`, puerto `2222`; en una VPS, a su IP, puerto `22`), y que el contenedor no tiene `systemd`, así que los servicios se manejan con `service` en lugar de `systemctl`.

---

## 1. La «máquina remota»: un contenedor Debian

Un contenedor de Docker es como una pequeña máquina Linux completa. Necesitamos Docker instalado (comprobamos con `docker -v`) y un cliente SSH.

```bash
docker run -d --name maquina-debian -p 2222:22 debian:trixie sleep infinity
```

| Parte | Significado |
|---|---|
| `-d` | Ejecutar en segundo plano |
| `--name maquina-debian` | Nombre para referirnos a ella |
| `-p 2222:22` | Publica el puerto 22 de la máquina en el 2222 de tu equipo |
| `debian:trixie` | Imagen oficial de Debian 13 |
| `sleep infinity` | Mantiene el contenedor encendido |

<br>

![Descarga de la imagen y creación del contenedor](img/1.png)

Entraremos en la consola de la máquina y comprobaremos cuál es:

```bash
docker exec -it maquina-debian bash
cat /etc/os-release
```
<br>

![Consola del contenedor con Debian 13 (trixie)](img/2.png)

IMPORTANTE: Necesitamos un acceso rescate. Dejaremos esta terminal abierta. Es como tener acceso físico a la máquina: aunque el SSH falle, desde aquí siempre podremos arreglarlo.

---