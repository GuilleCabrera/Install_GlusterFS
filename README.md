# Install_GlusterFS


GlusterFS agrega varios servidores de almacenamiento a través de Ethernet o la interconexión Infiniband RDMA en un gran sistema de archivos de red paralelos. Es un software libre, con algunas partes licenciadas bajo la Licencia Pública General de GNU (GPL) v3, mientras que otras tienen licencia dual bajo GPL v2 o la Licencia Pública General Menor (LGPL) v3. GlusterFS se basa en un diseño de espacio de usuario apilable.

GlusterFS tiene un componente de cliente y servidor. Los servidores generalmente se implementan como ladrillos de almacenamiento, con cada servidor ejecutando un daemon glusterfsd para exportar un sistema de archivos local como un volumen. El proceso del cliente glusterfs, que se conecta a servidores con un protocolo personalizado sobre TCP / IP, InfiniBand o Sockets Direct Protocol, crea volúmenes virtuales compuestos desde múltiples servidores remotos utilizando traductores apilables. De forma predeterminada, los archivos se almacenan enteros, pero también se admite la distribución de archivos en múltiples volúmenes remotos. El servidor final puede montar el volumen final utilizando su propio protocolo nativo a través del mecanismo FUSE, utilizando el protocolo NFS v3 utilizando un traductor servidor integrado, o accediendo a través de la biblioteca del cliente gfapi. Los montajes de protocolo nativo pueden reexportarse, por ej. a través del servidor kernel NFSv4, SAMBA, o el protocolo de almacenamiento basado en objetos OpenStack (Swift) utilizando el traductor "UFO" (Unified File and Object).

Por favor, lea estos términos de deslumbramiento:

- brick	: El ladrillo es el sistema de archivos de almacenamiento que se ha asignado a un volumen. Por ejemplo, datos en el servidor
- client : La máquina que monta el volumen (esto también puede ser un servidor).
- servidor : La máquina (física o virtual o bare metal) que aloja el sistema de archivos real en el que se almacenarán los datos.
- volumen : Un volumen es una colección lógica de ladrillos donde cada ladrillo es un directorio de exportación en un servidor. Un volumen puede ser de varios tipos y puede crear cualquiera de ellos en el grupo de almacenamiento para un solo volumen.

Distributed : los volúmenes distribuidos distribuyen archivos por los ladrillos del volumen. Puede usar volúmenes distribuidos cuando el requisito es escalar el almacenamiento y la redundancia no es importante o la proporcionan otras capas de hardware / software.

Replicated : los volúmenes replicados replican los archivos en los ladrillos del volumen. Puede usar volúmenes replicados en entornos donde la alta disponibilidad y la alta confiabilidad son críticas.

Striped : el volumen rayado raya los datos en los ladrillos del volumen. Para obtener los mejores resultados, debe usar volúmenes seccionados solo en entornos de alta concurrencia que accedan a archivos muy grandes.

Para nuestro laboratorio contaremos con 3 VM con CentOS 7.5:

- test-gluster01
- test-gluster02
- test-gluster03

Y en cada una de ellas configuraremos los repositorios de EPEL y de Gluster 4.1

- EPEL

```
[root@test-gluster01 ~]# rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

- Gluster 4.1

```
[root@test-gluster01 ~]# cat /etc/yum.repos.d/glusterfs.repo
[gluster4.1]
name = Gluster 4.1
baseurl = http://mirror.centos.org/centos/$releasever/storage/$basearch/gluster-4.1/
gpgcheck = 0
enabled = 1
```


Al tener configurados estos repositorios, podemos proceder a instalar los packages nececesarios 

```
[root@test-gluster01 ~]# yum -y install glusterfs glusterfs-fuse glusterfs-server
...
Dependencies Resolved

=======================================================================================================================
 Package                               Arch                Version                       Repository               Size
=======================================================================================================================
Installing:
 glusterfs                             x86_64              4.1.2-1.el7                   gluster4.1              648 k
 glusterfs-fuse                        x86_64              4.1.2-1.el7                   gluster4.1              146 k
 glusterfs-server                      x86_64              4.1.2-1.el7                   gluster4.1              1.4 M
Installing for dependencies:
 attr                                  x86_64              2.4.46-13.el7                 CentOS7                  66 k
 glusterfs-api                         x86_64              4.1.2-1.el7                   gluster4.1              105 k
 glusterfs-cli                         x86_64              4.1.2-1.el7                   gluster4.1              202 k
 glusterfs-client-xlators              x86_64              4.1.2-1.el7                   gluster4.1              969 k
 glusterfs-libs                        x86_64              4.1.2-1.el7                   gluster4.1              411 k
 psmisc                                x86_64              22.20-15.el7                  CentOS7                 141 k
 userspace-rcu                         x86_64              0.10.0-3.el7                  gluster4.1               93 k

Transaction Summary
=======================================================================================================================
Install  3 Packages (+7 Dependent packages)
...
Installed:
  glusterfs.x86_64 0:4.1.2-1.el7     glusterfs-fuse.x86_64 0:4.1.2-1.el7     glusterfs-server.x86_64 0:4.1.2-1.el7

Dependency Installed:
  attr.x86_64 0:2.4.46-13.el7                   glusterfs-api.x86_64 0:4.1.2-1.el7  glusterfs-cli.x86_64 0:4.1.2-1.el7
  glusterfs-client-xlators.x86_64 0:4.1.2-1.el7 glusterfs-libs.x86_64 0:4.1.2-1.el7 psmisc.x86_64 0:22.20-15.el7
  userspace-rcu.x86_64 0:0.10.0-3.el7
```

Iniciamos su servicio y lo configuramos para que este inicie con el servidor:

```
[root@test-gluster01 ~]# systemctl start glusterd.service
[root@test-gluster01 ~]# systemctl enable glusterd.service
```

Esto replicamos en todos los nodos de la plataforma

Una vez configurados todos los nodos debemos relacionarlos entre ellos con el siguiente comando:

gluster peer probe <hostname>

```
[root@test-gluster01 ~]# gluster peer probe test-gluster02
peer probe: success.
[root@test-gluster01 ~]# gluster peer probe test-gluster03
peer probe: success.
```

Una vez agregados todos los nodos podemos ver su estado de salud con el comando:

gluster pool list

```
[root@test-gluster01 ~]#  gluster pool list
UUID                                    Hostname        State
e45352cf-83fc-461d-be1b-93bd01905e62    test-gluster02  Connected
e2fc3482-8d80-46c7-bf34-5be4d97524ea    test-gluster03  Connected
9bc95ebe-4c4f-4b84-a948-ab676972bd25    localhost       Connected
[root@test-gluster01 ~]#
```


- Creando un Volumen Distributed

```
[root@test-gluster01 ~]# gluster volume create distribuido test-gluster01:/storage01/distribuido  test-gluster02:/storage01/distribuido  test-gluster03:/storage01/distribuido  force
volume create: distribuido: success: please start the volume to access data
```

Iniciamos Volumen

```
[root@test-gluster01 ~]# gluster volume start distribuido
volume start: distribuido: success
```

Revisamos la configuración del Volumen:

```
[root@test-gluster01 ~]# gluster volume info distribuido

Volume Name: distribuido
Type: Distribute
Volume ID: 4227163c-7d65-4499-9d71-d33cad4f51f7
Status: Started
Snapshot Count: 0
Number of Bricks: 3
Transport-type: tcp
Bricks:
Brick1: test-gluster01:/storage01/distribuido
Brick2: test-gluster02:/storage01/distribuido
Brick3: test-gluster03:/storage01/distribuido
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
```

```
[root@test-gluster01 ~]# gluster volume status
Status of volume: distribuido
Gluster process                             TCP Port  RDMA Port  Online  Pid
------------------------------------------------------------------------------
Brick test-gluster01:/storage01/distribuido 49152     0          Y       17979
Brick test-gluster02:/storage01/distribuido 49153     0          Y       5710
Brick test-gluster03:/storage01/distribuido 49152     0          Y       29329

Task Status of Volume distribuido
------------------------------------------------------------------------------
There are no active volume tasks

````