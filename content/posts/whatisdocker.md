+++
title = 'QUE ES DOCKER?'
+++

# Que es Docker?

### Primero, caso de uso

- Imagina que estas armando un proyecto con un equipo
- Supongamos que este proyecto usa una version especifica de python
- Cada miembro del equipo tiene que configurar todo a mano (dependencias, variables de entorno)
- Esto trae una cantidad variada de problemas
  - "A vos no te anda, en mi pc anda bien"
  - Si tenemos varios proyectos tenemos que invertir mucho tiempo en configurarlos

### Para poder evitar esto

### Usamos Containers

Una definicion basica y por arriba de que es un container es "un container es una carpeta o un paquete con todo lo que tu aplicacion necesita para funcionar"

Gracias a un container no nos tenemos que preocupar por la configuracion del proyecto, ya que todo lo que necesita esta dentro del container

Lo unico que necesitariamos es algo para manipular y administrar estos containers, que es donde aparece

## Docker

Docker es/tiene/puede:

- Tiene una gran comunidad por lo que se cuenta con herramientas y librerias disponibles (dockerhub)
  - Una herramienta para administrar containers
  - Una de las herramientas fundamentales en el area de DevOps
  - Facilita los procesos de CI/CD al proveer entornos para cada stage/etapa?
  - los containers usan menos recursos que una maquina virtual (despues se amplia)
  - Las aplicaciones corren igual sin importar en que maquina esten (OS agnostic)
  - Usa menos espacio ya que usa un file system basado en layers (se amplia despues)
  - ...

# Como se instala Docker?
En la pagina oficial de docker esta como instalar docker engine en distintas distribuciones de linux
https://docs.docker.com/engine/install/[DISTRO]/

En el caso de windows se puede instalar directamente docker desktop (docker engine con una UI)

una vez que terminamos de instalar, para verificar que ande todo bien corre este comando $ docker run hello-world

# Ya muestra el maldito container

Antes de llegar al container vamos con el dockerfile hace la imagen de la imagen sale el container
Dockerfile ------> Imagen --------> Container

El dockerfile es un archivo de texto (yaml) que tiene las instrucciones de como se va a hacer la imagen, es como armar un molde

que le hago al dockerfile para que haga la imagen?
corre este comando $ docker build -t nombre-de-la-imagen ./directorio-del-dockerfile

la imagen seria el molde, de la imagen sale un container (puede salir mas de uno)

$ docker run nombre-imagen


# Anatomia de un Dockerfile
basicamente un dockerfile describe los pasos necesarios para crear una imagen, por lo general va en el directorio root de la app.

cada linea del dockerfile crea una nueva layer de una imagen que docker la va a guardar, basicamente cuando hagas imagenes nuevas docker solo necesita crear layers de imagenes que varien de la anterior


FROM node:0.10 ---> imagen base

MAINTAINER Anna Doe <anna@example.com> --> metadata
LABEL "rating"="Five Stars" "class"="First Class" --> metadata
USER root (por mas que los containers provean aislamiento del os siguen corriendo en el kernel del host, en los containers de produccion siempre se debe correr como un usuario no privilegiado)

-- Setear variables de entorno para simplificar el proceso de crear la imagen, manteniendo el dockefile mas simple y siguiendo el principio DRY(dont repeat yourself)

ENV AP /data/app
ENV SCPATH /etc/supervisor/conf.d



RUN apt-get -y update --> esto va a hacer que el build sea mas lento, si podes conseguir una imagen que este actualizada

#### The daemons (esto instala algunas dependencias que necesito usar, tambien forma la estructura archivos que necesito)

RUN apt-get -y install supervisor
RUN mkdir -p /var/log/supervisor

#### Supervisor Configuration
-- Add sirve para copiar archivos del filesystem local a tu imagen (una vez que haga la imagen puedo usarla sin tener accesso al file system original, ya que los archivos fueron copiados)
ADD ./supervisord/conf.d/\* $SCPATH/

#### Application Code

El orden de los comandos va a afectar el tiempo de ejecucion, lo optimo es poner los que cambian entre cada imagen al fondo

ADD _.js_ $AP/
WORKDIR $AP
RUN npm install

CMD corre el comando que inicia el proceso que quiero correr dentro del container

CMD ["supervisord", "-n"]

Lo que se considera como best practice es correr un solo proceso en un container

## CONTAINERS VS VIRTUAL MACHINES

Si muy lindo el container, pero puedo hacer una maquina virtual y es lo mismo

Por que los containers y las maquinas virtuales no son lo mismo?
Porque si fueran lo mismo se llamarian igual, chau.

Bueno, un viaje por distintas epocas de desarrollo de software
![K8s Virtualization](https://kubernetes.io/images/docs/Container_Evolution.svg)

En la etapa tradicional se usaban servidores fisicos, no podemos definir recursos para distintas partes del servidor, en el caso de correr 2 aplicaciones en el mismo servidor, si una toma muchos recursos hacia caer la otra, la solucion es usar varios servidores, pero el problema de esto es el alto precio de escalar servidores fisicos y que hago con los recursos no utilizados. 

Deploys virtualizados: Otra solucion fue correr maquinas virtuales en un servidor, la virtualizacion permite aislar y asegurar los procesos ya que cada una corre su propio sistema operativo, por lo que el resto no pueden acceder.
Virtualizando optimizamos el uso de recursos en el servidor fisico, esto escala mejor ya que puedo usar un grupo de servidores fisicos con maquinas virtuales desechables

Containers: a diferencia de las maquinas virtuales comparten sistema operativo entre las aplicaciones, tienen su propio filesystem, cpu y memoria. Lo que los hace muy utiles es el ser portables a lo largo de sistemas operativos y distintos cloud providers

## Como se aislan los procesos en containers?

    Namespaces: Docker creates separate namespaces for the container, isolating its process IDs, network interfaces, mounts, IPC resources, hostname, and user/group IDs. This gives the container its own isolated environment.



Kernel Namespaces, esto es una feature de linux
Docker usa:
 - PID (process ID): cada container tiene su propio set de process ids, dentro del container los procesos tienen distinto PID que en el host, esto previene al container de interactuar con procesos fuera de el
 - Network: Aisla las interfaces de red, ips, routing tables. Cada container tiene su propio network stack, por lo que tiene su propia ip y configuracion de red, distinta del host.

 - Mount: Esto es una forma de montar y desmontar filesystems sin afectar al host

 - UTS: para aislar el hostname y domain name, util para las configuraciones de red

 - User: Esto permite el mapeo de usuarios y group IDS dentro del container, esto provee seguridad al no darles acceso directo a los privilegios de root del host (mucho muy importante)

En docker: se crean namespaces separados para el container, lo que crea su propio entorno aislado

## Que se encarga de los recursos que usa cada container?

Control groupc (cgroups), tambien son una feature del kernel de linux, lo que aisla los recursos dentro de cada container (cpu, memoria y operaciones sobre el disco)
En el caso de docker, pone los procesos del container en cgroups, que controlan y limitan los procesos del container, esto asegura la buena performance.



# En caso de que quieras ver con mas profundidad estos ultimos temas
https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504
https://medium.com/@saschagrunert demystifying-containers-part-ii-container-runtimes-e363aa378f25
https://medium.com/@saschagrunert/demystifying-containers-part-iii-container-images-244865de6fef