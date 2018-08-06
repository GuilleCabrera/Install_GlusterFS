# Install_GlusterFS


GlusterFS agrega varios servidores de almacenamiento a través de Ethernet o la interconexión Infiniband RDMA en un gran sistema de archivos de red paralelos. Es un software libre, con algunas partes licenciadas bajo la Licencia Pública General de GNU (GPL) v3, mientras que otras tienen licencia dual bajo GPL v2 o la Licencia Pública General Menor (LGPL) v3. GlusterFS se basa en un diseño de espacio de usuario apilable.

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

Esto replicamos en todos los nodos de la plataforma
