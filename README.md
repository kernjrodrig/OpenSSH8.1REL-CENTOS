
# OpenSSH8.1REL-CENTOS

Upgrade OpenSSH 8.1 en  Centos/Redhat 7.7 / 7.9 /8 /8.6 

# Compilar OPENSSH8.1 en REDHAT /CENTOS 8 version portable



Upgrade OpenSSH Centos/Redhat 8.6  6/ 7.9 /8

PARTE 1

1- Primero debe instalar algunas dependencias, como herramientas de desarrollo o elementos esenciales de compilación y los demás paquetes necesarios.

        yum groupinstall "Development Tools"

        yum install zlib-devel openssl-devel

2- Descargar el openssh V8.#

        wget -c https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-8.1p1.tar.gz


3- Descomprir y entran en el directiorio

        tar -xzf openssh-8.0p1.tar.gz
        cd openssh-8.0p1/

4- Compilar e instalar SSH 

        ./configure --with-md5-passwords --with-pam --with-selinux --with-privsep-path=/var/lib/sshd/ --sysconfdir=/etc/ssh

        make

        make install

5- Una vez que haya instalado OpenSSH, reinicie SSH y verifique la versión de OpenSSH

        systemctl restart sshd.service

        ssh -V



troubleshooting:


Error post update.

sshd -t

        root@mysql5-slave proj]# sshd -t
        @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
        @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
        @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
        Permissions 0640 for '/etc/ssh/ssh_host_rsa_key' are too open.
        It is required that your private key files are NOT accessible by others.
        This private key will be ignored.
        key_load_private: bad permissions
        Could not load host key: /etc/ssh/ssh_host_rsa_key
        @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
        @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
        @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
        Permissions 0640 for '/etc/ssh/ssh_host_ecdsa_key' are too open.
        It is required that your private key files are NOT accessible by others.
        This private key will be ignored.
        key_load_private: bad permissions
        Could not load host key: /etc/ssh/ssh_host_ecdsa_key
        @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
        @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
        @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
        Permissions 0640 for '/etc/ssh/ssh_host_ed25519_key' are too open.
        It is required that your private key files are NOT accessible by others.
        This private key will be ignored.
        key_load_private: bad permissions
        Could not load host key: /etc/ssh/ssh_host_ed25519_key
        sshd: no hostkeys available -- exiting.


Para solucionar esto solo debermos dar permisos a las keys


chmod 600 /etc/ssh/ssh_host_rsa_key /etc/ssh/ssh_host_ecdsa_key /etc/ssh/ssh_host_ed25519_key


Error 2:

        @@@@@@@@@@@@@
        @ WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! @
        @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
        IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
        Someone could be eavesdropping on you right now (man-in-the-middle attack)!
        It is also possible that a host key has just been changed.
        The fingerprint for the ED25519 key sent by the remote host is
        SHA256:s+E4dL4Ahj5hFVcPn1YErU+/77q4WOKO4qhNqK6siuQ.
        Please contact your system administrator.
        Add correct host key in /root/.ssh/known_hosts to get rid of this message.
        Offending ED25519 key in /root/.ssh/known_hosts:6
        ED25519 host key for 192.168.xx.xxx has changed and you have requested strict ch
        Host key verification failed.


Borrar la línea /root/.ssh/known_hosts:6 


PARTE 2:

Luego de compilar el servicio vamos a generar un demonio y desisntalar la version anterior para no generar conflitos dentro del sistema.


1- para diferenciar los 2 serviciones:

        /usr/local/sbin/sshd -V
        ps -ef | grep ssh
        /usr/sbin/sshd -D

        /usr/sbin/sshd -V				 version 7.4 OLD
        /root/openssh/openssh-8.1p1/sshd -V		 version 8.1 Versioon nueva 



2- Vamos a copiar el sshd 

        cp /root/openssh/openssh-8.0p1/sshd /usr/sbin/sshd
        ## Si este comando falla  hacer lo siguiente:
        cd /usr/local/sbin/
        cp sshd /usr/sbin/sshd

Si no funciona forzarlo con -f



3- Creamos un script para levantar o bajar el servicio nuevo. (Esta parte pueden manejarlo como prefieran)


        cd /etc/init.d
        vim sshd

/etc/init.d/sshd

        #! /bin/sh
        ###### start/stop the secure shell daemon
        ###### chkconfig: 2345 95 05
        case "$1" in
        'start')

        # Start the ssh daemon
        if [ -x /usr/local/sbin/sshd ]; then
                echo "starting SSHD daemon"
                /usr/local/sbin/sshd -f /usr/local/etc/sshd_config &
        fi
        ;;

        'stop')
        # Stop the ssh daemon
        ######      /usr/bin/pkill -x sshd
        PIDSSHD=`ps -ef | grep "/sbin/sshd" | grep -v grep | awk '{print $2}'`
        /usr/bin/kill -9 $PIDSSHD
        ;;
        *)
        echo "usage: /etc/init.d/sshd {start|stop}"
        ;;
        esac


        chmod 755 sshd
        cd ../rc3.d
        ln -s ../init.d/sshd S60sshd


4- Probemos que el servicio levante.

        systemctl stop sshd <- old ssh 

        /etc/init.d/sshd start <- New


5- ahora podemos pasar a desisntalar la version anterior. ( hay que tener en cuenta varios cosas antes de realizar este paso.) 

a) Solo hay que desistalar el openssh-server-7.4p1-21.0.1.el7.x86_64 o la version anterior que posea el sistema operativo.

Si desistanal todos los servicios se rompoe el openssh.

b) Al borrar el openssh-server se borran un archivo pam necesarios para el correcto funcionamiento del mismo.

/etc/pam.d/sshd

c) En la parte 1 del compilado observamos que el openssh se compula en el directorio /usr/local/etc/ Por ende debemos copiar todo el contenido de /etc/ssh/ para no peder las configuraciones previas del servicio y pueda funcionar sin problemas.




5.1- Respaldar los archivos /etc/pam.d/sshd 


6- todo los arhivos de /etc/ssh/ copiarlos o moverlos a /usr/local/etc/ 

7- modificar el Include de ssh_config


        Include /usr/local/etc/ssh_config.d/*.conf


7.1- vamos a borrar el openssh-server
rpm -qa | grep opens
yum remove openssh-server-7.4p1-21.0.1.el7.x86_64

No rebootear la maquina despues de hacer este paso ya que no podran entrar al servidor ( puede fallar en inid)

7.2- Restaurar el respaldar los archivos /etc/pam.d/sshd (Si no lo hiciste te dejo los datos del archivo)

        #%PAM-1.0
        auth       substack     password-auth
        auth       include      postlogin
        account    required     pam_sepermit.so
        account    required     pam_nologin.so
        account    include      password-auth
        password   include      password-auth
        # pam_selinux.so close should be the first session rule
        session    required     pam_selinux.so close
        session    required     pam_loginuid.so
        # pam_selinux.so open should only be followed by sessions to be executed in the user context
        session    required     pam_selinux.so open env_params
        session    required     pam_namespace.so
        session    optional     pam_keyinit.so force revoke
        session    optional     pam_motd.so
        session    include      password-auth
        session    include      postlogin



8- Crear demonio para sshd:

En el directorio: /lib/systemd/system/

Creamos el archive sshd.service

En caso de que este ya exista se hace un respaldo del mismo.



Dentro del archive sshd.service 

        [Unit]
        Description=sshd
        After=network.target
        StartLimitIntervalSec=0

        [Service]
        Type=simple
        Restart=always
        RestartSec=1
        User=root
        ExecStart=/usr/local/sbin/sshd -f /usr/local/etc/sshd_config

        [Install]
        WantedBy=multi-user.target


systemctl daemon-reload

Ya con esto Podemos controlar el demon del sshd con systemctl

Por ultimos habilitamos el demon para que inicie automaticamente al bootear:

systemctl enable sshd


Una vez finalizado estos passo podemos rebootear el sistema.





Perdon por el desorden de pasos, pronto lo organizare mejor.
Por otro lado Esto funciona con centos/redhat/oracle linux 7.7 8.0 8.4 8.6 no se si funcionan versiones posteriores a 8.1 del openssh.






## Installation

Install my-project with npm

```bash
  npm install my-project
  cd my-project
```
    