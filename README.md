TEST DEVOPS

                                          Instalacion de kubernetes
                                          
          Instalar Kubernetes Cluster en CentOS 8 con Ansible
      
      1.- En su CentOS 8 Master-Node , configure el nombre de host del sistema y actualice DNS en su archivo / etc / hosts 
            #hostnamectl set-hostname master-node
            
            editar el archivo HOSTS y declarar las IP de los nodos y Master y comprobar con PING para si el cambio funciona
            #vim /etc/host
            ejm:
           192.168.162.47 nodo maestro
           192.168.162.48 nodo-1 nodo-1
           192.168.162.49 nodo-2 nodo-2
           
           #ping 192.168.162.47
           
     2.-   desactive Selinux , ya que esto es necesario para permitir que los contenedores accedan al sistema de archivos host, que es              necesario para las redes de pod y otros servicios.
     
            #setenforce 0
           
         Para deshabilitarlo por completo, use el siguiente comando y reinicie
         
         #sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
         # reboot
         
     3.- Configure las reglas del firewall en los puertos.
     
         # firewall-cmd --permanent --add-port = 6443/tcp 
         # firewall-cmd --permanent --add-port = 2379-2380/tcp 
         # firewall-cmd --permanent --add-port = 10250/tcp 
         # firewall-cmd --permanent --add-port = 10251/tcp 
         # firewall-cmd --permanent --add-port = 10252/tcp 
         # firewall-cmd --permanent --add-port = 10255/tcp 
         # firewall -cmd –reload 
         # echo '1'> /proc/sys/net/bridge/bridge-nf-call-iptables
     
     4.- Instalar Docker en CentOS 8
     
     Habilita el Repositorio Docker-ce
          # dnf config-manager --add-repo = https: //download.docker.com/linux/centos/docker-ce.repo
          
     Instale Docker CE usando el comando dnf
          #  dnf install docker-ce --nobest -y
          
     inicie y habilite su servicio utilizando los siguientes comandos systemctl
          # systemctl enable docker
          # systemctl start docker
          
     Ejecute el siguiente comando para verificar la versión de Docker instalada
          # docker --version 
          
     Verifique y pruebe el motor Docker CE
          de prueba usemos: 
                #docker run hello-world
  Debera mostrar un mensaje como el de acontinuacion:
  
      " Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
                                                                                                          "
                                                                                                          
  
      Instalar el Docker Compose
      
      Docker compose se usa para vincular múltiples contenedores usando un solo comando. En otras palabras, Docker Compose es útil cuando necesitamos lanzar múltiples contenedores y estos contenedores dependen unos de otros.
      
      Ejecute los siguientes comandos para instalar docker compose en CentOS 8:
      
       # dnf install curl -y
       # curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose - $ (uname -s) - $ (uname -m) "-o /usr/local/bin/docker-compose
       
   Establecer el permiso ejecutable para docker-compose
        # chmod + x /usr/local/bin/docker-compose 
  
  Verifique la versión de redacción de Docker ejecutando el siguiente comando
          #docker-compose --version 
       

     5.- Instale Kubernetes (Kubeadm) en CentOS 8
     
          A continuación, deberá agregar repositorios Kubernetes manualmente, ya que no vienen instalados de forma predeterminada en CentOS 8 .
          
          # cat << EOF> /etc/yum.repos.d/kubernetes.repo 
          [kubernetes] 
          name = Kubernetes 
          baseurl = https: //packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64 
          enabled = 1 
          gpgcheck = 1 
          repo_gpgcheck = 1 
          gpgkey = https: //packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key .gpg 
          EOF
            
            
    Con el repositorio de paquetes ahora listo, puede continuar e instalar el paquete kubeadm.
          # dnf install kubeadm -y
          
    Cuando la instalación se complete con éxito, habilite e inicie el servicio.
          # systemctl enable kubelet 
          # systemctl start kubelet
          
   Crear el Master del Kubernete
          Debemos deshabilitar el intercambio para ejecutar el comando " kubeadm init ".
          #swapoff -a
   luego vas al archivo
          #vim /etc/fstab
    y comentas la linea: /dev/mapper/cl-swap     swap                    swap    defaults        0 0
    
   ahora procedemos a iniciar el kubeadm
          #kubeadm init
          
          
  A continuación, copie el siguiente comando y guárdelo en algún lugar, ya que este comando se ejecuta tambien en los nodos de trabajo
  
          #kubeadm join 192.168.162.47:6443 --token nu06lu.xrsux0ss0ixtnms5  --discovery-token-ca-cert-hash ha256: f996ea35r4353d342fdea2997a1cf8caeddafd6d4360d606dbc8231468778hj
          
          
Una vez que Kubernetes se haya inicializado correctamente, debe permitir que su usuario comience a usar el clúster. En nuestro escenario, utilizaremos el usuario root. También puede iniciar el clúster utilizando sudo user como se muestra.

ejecute para root:
          # mkdir -p $HOME/.kube 
          # cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
          # chown $(id -u):$(id -g) $HOME/.kube/config
         
ejecute para usuario habilitado en sudo:
          $mkdir -p $HOME/.kube 
          $ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
          $ sudo chown $(id -u):$(id -g) $HOME/.kube/config
          
  Ahora confirme que el comando kubectl está activado:
          #kubectl get node
  
  les debe mostrar algo como esto:
  
"  NAME     STATUS   ROLES    AGE     VERSION 
   master   Ready    master   5m19s   v1.18.2   "
   
   
   Configura tu red Pod
   
   La implementación del clúster de red es un proceso altamente flexible que depende de sus necesidades y hay muchas opciones disponibles. Como queremos que nuestra instalación sea lo más simple posible, utilizaremos el complemento Weavenet que no requiere ninguna configuración o código adicional y proporciona una dirección IP por pod, lo cual es excelente para nosotros. Si desea ver más opciones aqui les dejo el link:
   
   https://kubernetes.io/docs/concepts/cluster-administration/networking/
   
   Estos comandos serán importantes para obtener la configuración de la red de pod.
   
    # export kubever=$(kubectl version | base64 | tr -d '\n')
    # kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
    
    nos debe mostrat esto:
    
    "     [root@master etc]# kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
          serviceaccount/weave-net created
          clusterrole.rbac.authorization.k8s.io/weave-net created
          clusterrolebinding.rbac.authorization.k8s.io/weave-net created
          role.rbac.authorization.k8s.io/weave-net created
          rolebinding.rbac.authorization.k8s.io/weave-net created
          daemonset.apps/weave-net created                                                "
          
   
   y solo nos queda verificar de nuenvo el estatus del los nodos
   
            # kubectl get node
          
          
   NOTA: Para los NODOS y su configuracion solo hay que seguir los pasos anteriores, pero cambiando el nombre del HOSTNAME que seria en           este caso el nombre asignados para los nodos.y utlizando el comando para unir los nodos:
   
   
   
        # kubeadm join 192.168.162.47:6443 --token nu06lu.xrsux0ss0ixtnms5 --discovery-token-ca-cert-hash sha256: f996ea35r4353d342fdea2997a1cf8caeddafd6d4360d606dbc82314683478jj




    6.- Instalar Ansible
    
    habilitar el repositorio de ansible:
    
    El paquete Ansible no está disponible en el repositorio de paquetes predeterminado de CentOS 8. entonces necesitamos habilitar EPEL     Repository ejecutando el siguiente comando:
    
             $ sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y
             
    Una vez que el repositorio de epel esté habilitado, ejecute el siguiente comando dnf para instalar Ansible
    
            $ sudo dnf install ansible
            
    Una vez que el ansible se haya instalado correctamente, verifique su versión ejecutando el siguiente comando:
    
             $ sudo ansible --version
             
     debe mostrar algo asi:
     
      "           [root@master etc]# ansible --version
                  ansible 2.9.7
                  config file = None
                  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
                  ansible python module location = /root/.local/lib/python3.6/site-packages/ansible
                  executable location = /usr/local/bin/ansible
                  python version = 3.6.8 (default, Nov 21 2019, 19:31:34) [GCC 8.3.1 20190507 (Red Hat 8.3.1-4)]              "
                  
       
      
        Configuracion de Ansible
              vamos a añadir un grupo con nombre "webservers"  y la dirección IP del sistema en el archivo /etc/ansible/hosts:
              
              #vim /etc/ansible/hosts
              ... 
              [webservers] 
              192.168.162.47 (el master)
              192.168.162.48 (el nodo)
              ...


        Una vez que el archivo de inventario (/etc/ansible/hosts) se actualiza, intercambie las claves públicas ssh de su usuario          con sistemas remotos que forman parte del grupo de "webservers".


      Primero generemos la clave pública y privada de su usuario local usando el comando ssh-keygen:
  
            $ ssh-keygen
            
     Ahora intercambie la clave pública entre el servidor ansible y sus clientes usando el siguiente comando:
     
            $ ssh-copy-id pkumar@192.168.162.47
            $ ssh-copy-id pkumar@192.168.162.48
            
      Ahora intentemos un par de comandos Ansible, primero verifique la conectividad del servidor Ansible a sus clientes usando el             módulo ping:

            $ ansible -m ping "webservers"
            
     debe mostrarles algo como esto:
     
     
  "      [root@master ansible]# ansible -m ping "webservers"
          192.168.162.131 | SUCCESS => {
          "ansible_facts": {
          "discovered_interpreter_python": "/usr/libexec/platform-python"
           },
           "changed": false,
           "ping": "pong"
          }                                   
          
           192.168.162.129 | SUCCESS => {
          "ansible_facts": {
          "discovered_interpreter_python": "/usr/libexec/platform-python"
           },
           "changed": false,
           "ping": "pong"
          }                                                                     "


    Use el siguiente comando ansible para enumerar todos los hosts del archivo de inventario:
    
            $ansible all -i /etc/ansible/hosts --list-hosts 
            
     muestra algo como esto:
     
          "       root@master ansible]# ansible all -i /etc/ansible/hosts --list-hosts 
                  hosts (3):
                  ...
                  192.168.162.131
                  192.168.162.129                                                             "
                  
                  
       7.- procedemos a instalar Kubespray
       
       # CLONAR REPOSITORIO KUBESPRAY 2.11
      $ git clone -b release-2.11 https://github.com/kubernetes-sigs/kubespray/

      # POSICIONARSE EN REPOSITORIO CLONADO
          $ cd kubespray

          # CREAR CARPETA ESPECIFICA DE CLUSTER A CREAR
        $ cp -rfp inventory/sample inventory/name-cluster

          # INSTALA DEPENDENCIAS
          $ pip install -r requirements.txt

          # AJUSTAR VARIABLES DE CLUSTER
          $ vi inventory/name-cluster/group_vars/k8s-cluster/k8s-cluster.yml

            # SE MODIFICA NOMBRE DEL CLUSTER
             cluster_name: Name-cluster

             kube_version: v1.15.6

          # SE ESPECIFICA LOS MASTER Y NODOS
          $ vi inventory/name-cluster/hosts.ini

           [all]
    master1 ansible_host=192.168.162.131 ip=192.168.162.131 etcd_member_name=etcd01
    nodo1 ansible_host=192.168.162.129 ip=192.168.162.129

          [kube-master]
          master1

          [etcd]
          master1

          [kube-node]
          nodo1

          [k8s-cluster:children]
          kube-master
          kube-node

        # VERSION DE ANSIBLE
        $ ansible --version

        # SI LA VERSION DE ANSIBLE ES < 2.7
        $ yum update -y ansible

        # SE INSTALA COMPLEMENTO ADICIONAL
        $ yum install python-netaddr

        # INSTALAR KUBERNETES (comprueba si esta funcionando el cluster)
        $ ansible-playbook -i inventory/cluster-bci/hosts.ini cluster.yml (debe estar logueado ssh)
        
   ya debeira estar funcionando el cluster
   
   
   
   8.- Desplegar la aplicacion Docker
   
   Compruebe que inicialmente no hay ningún contenedor creado (la opción -a hace que se muestren también los contenedores detenidos, sin ella se muestran sólo los contenedor que estén en marcha)
        
          #docker ps -a
          
  o también
  
          #docker container ls -a
          
   Compruebe que inicialmente tampoco disponemos de ninguna imagen
   
            #docker image ls
            
    debe aparecerte algo similar a esto:
    
    "          [root@master ansible]# docker image ls
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
testdevops/newimage                  latest              aefe5f4fb354        23 hours ago        118MB          "



Docker crea los contenedores a partir de imágenes locales (ya descargadas), pero si al crear el contenedor no se dispone de la imagen local, Docker descarga la imagen de su repositorio.

La orden más simple para crear un contenedor es:
        
          #docker run IMAGEN

Cree un contenedor con la aplicación de ejemplo hello-world. La imagen de este contenedor se llama hello-world

          #docker run hello-world


Como no tenemos todavía la imagen en nuestro ordenador, Docker descarga la imagen, crea el contenedor y lo pone en marcha. En este caso, la aplicación que contiene el contenedor hello-world simplemente escribe un mensaje de salida al arrancar e inmediatamente se detiene el contenedor. La respuesta será similar a esta:

   "     #Unable to find image 'hello-world:latest' locally
          latest: Pulling from library/hello-world
          1b930d010525: Pull complete
          Digest: sha256:4fe721ccc2e8dc7362278a29dc660d833570ec2682f4e4194f4ee23e415e1064
          Status: Download newer image from hello-wolrd:latest

          Hello from Docker!
          This message shows that your installation appears to be working correctly.            "
          
          
          
   Si listamos ahora las imágenes existentes
   
          #docker image ls
          REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
          hello-world         latest              fce209e99eb9        4 weeks ago         1.84 kB
    
    
   Si listamos ahora los contenedores existentes 
   
          #docker ps -a
          CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS
          PORTS               NAMES
          614B4c431ffa        hello-world         "/hello"            2 minutes ago       Exited (0) 2 minutes
          ago                 hopeful_elbakyan
        
        
        
 
Los contenedores se pueden destruir mediante el comando rm, haciendo referencia a ellos mediante su nombre o su id.

                    #docker rm hopefukl_elbakyan
                    hopefukl_elbakyan
                    
                    
                    
                    
  #####    AHORA CREAMOS NUESTRA IMAGEN #####
       
      
 1.- Crear un contenedor que contenga un servidor a partir de la imagen bitnami/apache
 
          #docker run -d -P --name=testdevops testdevops/newimage
          44c9e89bdfd24fa623774b3e774b0fb0efa107f752af5161b1d6b925330a82f6
          
  2.- Consulte el puerto del host utilizado por el contenedor.
  
            #docker ps -a     
            
      se mostrará el contenedor con el nombre que hemos indicado:
     
     CONTAINER ID        IMAGE               COMMAND                  CREATED                  STATUS
      PORTS                                              NAMES
    
    cbfc18d4f31f        testdevops/newimage     "/opt/bitnami/script…"   23 hours ago        Up 14 hours                 0.0.0.0:32769->8080/tcp, 0.0.0.0:32768->8443/tcp       testdevops
      
      
      
   3.- Abra en el navegador la página inicial del contenedor y compruebe que se muestra una página que dice.
   
                                  "It works!".
   
   
    4.- Modificar la página inicial del contenedor
    
    Cree la nueva página index.html
    
            #vim index.html
    
   "       <!DOCTYPE html>
           <html lang="es">
           <head>
           <meta charset="utf-8">
           <title>Apache en Docker</title>
           <meta name="viewport" content="width=device-width, initial-scale=1.0">
           </head>

           <body>
           <h1>Test_Devops</h1>
           </body>
           </html>                                      "
           
           
     o tambien
     
      "     <h1>Test_Devops</h1>         "
  
  
  5.- Entre en la shell del contenedor para averiguar la ubicación de la página inicial:
  
           #docker exec -it apache-2 /bin/bash
        
           Se abrirá una shell en el contenedor. El DocumentRoot de Apache está en el directorio /opt/bitnami/apache/htdocs
        
           #I have no name!@c5041d12aabd:/app$ cd /opt/bitmani/apache/htdocs
        
   
   6.- En ese directorio se encuentra el fichero index.html que queremos modificar
   
             #cat index.html
             <html><body><h1>It works!</h1></body></html>
        
        
             salga de la shell del contenedor
             #I have no name!@c5041d12aabd:/opt/bitnami/apache/htdocs$ exit
        
   7.- Copie el fichero index.html en el contenedor:
   
             #sudo docker cp index.html apache-2:/opt/bitnami/apache/htdocs/index.html
             
    
   8.- Abra en el navegador la página inicial del contenedor y compruebe que se ha modificado.


          192.168.162.131:(numero de puerto)
          
                                  "Test Devops"
                                  
                                  
    9.- Para ver el numero de puesto que posee la imagen ejecutamos:
    
            #docker port "nombre de la imagen"
                                  
                                  
NOTA:  Si queremos cambiar la página inicial, la forma correcta de hacerlo en Docker es crear una nueva imagen que incluya la página modificada, de manera que cada vez que se cree el contenedor, la página inicial sea la modificada.


      
   1.- Cree un directorio que contendrá el Dockerfile
            #mkdir testdevops
            
   2.- Copie el fichero index.html creado anteriormente
            #mv index.html testdevops
            
   3.-  Entre en el directorio y cree un fichero Dockerfile
   
            #cd testdevops
            #sudo nano Dockerfile
  
   4.- El contenido del Dockerfile puede ser el siguiente:
   
   "       FROM bitnami/apache
          COPY index.html /opt/bitnami/apache/htdocs/index.html    "
          
          
    5.- Genere la nueva imagen.

El último argumento (y el único imprescindible) es el nombre del archivo Dockerfile que tiene que utilizar para generar la imagen. Como en este caso se encuentra en el mismo directorio y tiene el nombre predeterminado Dockerfile, se puede escribir simplemente punto (.).

Para indicar el nombre de la imagen se debe añadir la opción -t. El nombre de la imagen debe seguir el patrón nombre-de-usuario/nombre-de-imagen. Si la imagen sólo se va a utilizar localmente, el nombre de usuario y de la imagen pueden ser cualquier palabra.
          
          #docker build -t prueba/tesdeops
          
    
    6.- Cree un contenedor a partir de la nueva imagen
    
          #docker run -d -P --name=newimagen prueba/testdevops
  
  
  
    7.- Abra en el navegador la página inicial del contenedor y compruebe que se ha modificado
 
                192.168.162.131:port
                
                                      "Test Devops"


   
   
   



    

 
      

           
           
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
            
      
