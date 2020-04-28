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
     
         # firewall-cmd --permanent --add-port = 6443 / tcp 
         # firewall-cmd --permanent --add-port = 2379-2380 / tcp 
         # firewall-cmd --permanent --add-port = 10250 / tcp 
         # firewall-cmd --permanent --add-port = 10251 / tcp 
         # firewall-cmd --permanent --add-port = 10252 / tcp 
         # firewall-cmd --permanent --add-port = 10255 / tcp 
         # firewall -cmd –reload 
         # echo '1'> / proc / sys / net / bridge / bridge-nf-call-iptables
     
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
               
    ! [imagen]_(https://www.linuxtechi.com/wp-content/uploads/2019/12/hello-world-container-centos8.png)
     
    
       
       
         
  
     
            
           
           
            
            
      
