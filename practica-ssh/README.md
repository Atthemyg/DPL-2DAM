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

## 2. Instalar y arrancar el servidor SSH

Dentro de la consola del contenedor:

```bash
apt update
apt install openssh-server sudo
```

Pulsaremos Enter cuando pregunte `Do you want to continue?`.

![Instalación de openssh-server y sudo](img/3.png)

Arrancamos el servicio y comprobamos su estado:

```bash
service ssh start
service ssh status
```

Debe responder `sshd is running.`

![Servicio SSH arrancado](img/4.png)

IMPORTANTE: ¿Por qué hay que arrancarlo a mano? En una Debian normal el servicio arranca solo al instalarse. En el contenedor no hay `systemd` y Docker impide que `apt` arranque servicios, así que se hace manualmente.

| Acción | Máquina real (systemd) | Contenedor |
|---|---|---|
| Arrancar | `sudo systemctl start ssh` | `service ssh start` |
| Ver estado | `sudo systemctl status ssh` | `service ssh status` |
| Recargar configuración | `sudo systemctl reload ssh` | `service ssh reload` |

---

## 3. Crear un usuario

No nos conectaremos como `root`: Debian lo impide por defecto con contraseña y es más seguro usar un usuario normal.

```bash
adduser usuario
```

Pide una contraseña (dos veces) y algunos datos opcionales que dejaremos vacíos con Enter. Al final confirmamos con `Y`.

![Creación del usuario](img/6.png)

Le daremos permisos de administración:

```bash
adduser usuario sudo
```

![Usuario añadido al grupo sudo](img/7.png)

---