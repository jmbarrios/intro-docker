---
theme: gaia
footer: JMB @ CONABIO
---
<!-- _class: lead gaia -->
![docker-logo bg left:40% 60%](https://www.docker.com/assets/logo-files/Docker-Logo-White-RGB_Moby.png)

# Docker

¿Qué es y cómo usarlo?

---

# ¿Qué es Docker?

- Docker es una aplicación para correr contenedores.
- Sirve para correr aplicaciones de manera aislada.
- Parecida a una máquina virtual.
- Es muy útil para hacer despliegue de aplicaciones.

--- 

![container bg height: 70%](https://www.docker.com/sites/default/files/d8/2018-11/docker-containerized-appliction-blue-border_2.png)

![vm bg height: 70%](https://www.docker.com/sites/default/files/d8/2018-11/container-vm-whatcontainer_2.png)

---

# Imagen de Docker

Es una fotografía del sistema que quieres ejecutar para correr tu aplicación:

  - SO,
  - dependencias,
  - código de la aplicación.

---

# Contenedor

Un contenedor es un proceso que ejecuta una imagen. 

---

![map-ports bg left 80%](./img/port_mapping.png)

# Mapeo de puertos

Asociamos el puerto **8080** de la máquina host al puerto **80** que expone el contenedor de NGNIX

```
docker run -p 8080:80 nginx:latest
```
---

![map-ports bg left 80%](./img/bind_volume.png)

### Compartir archivos entre _host_ y _container_

```
docker run -d --name website -p 8080:80 \
  -v /mi-ruta/folder/website:/usr/share/nginx/html:ro \
  nginx:latest
```
---

![map-ports bg left 80%](./img/container_volume.png)

### Compartir archivos entre _containers_

```
docker run -d --name website2 -p 8081:80 \
  --volumes-from website  \
  nginx:latest
```
---
# Dockerfile

- Es una manera de crear una imagen de Docker

```
FROM alpine:3
CMD ["sh", "-c", "echo 'Hola desde mi contenedor '$HOSTNAME"]
```
---
# _Tags_, versionamiento y etiquetado

- Permite tener un control sobre la versión de la imagen
- Evita que haya cambios que modifiquen la aplicación