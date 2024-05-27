# ApuntesSI
### Resumen del Texto sobre Configuración de la Red y Iptables

#### 1.1 Configuración de la Red

**Interfaces de red:**  
Configurar interfaces de red es el primer paso al implementar una red. Se puede hacer manualmente o mediante un servidor DHCP. Para redes cableadas, se usa el comando `ifconfig` para asignar direcciones IP. Para redes inalámbricas, se utilizan `iwconfig` o herramientas gráficas.

**Ficheros de configuración:**  
Para guardar configuraciones persistentes, se editan archivos como `/etc/network/interfaces` para interfaces de red y `/etc/hostname` para el nombre del equipo. El archivo `/etc/resolv.conf` se usa para servidores DNS.

**Comprobación:**  
Se verifica la configuración usando comandos como `ping` para probar la conexión a Internet y herramientas gráficas disponibles en GNU/Linux.

#### 1.2 Iptables

**Evolución:**  
Iptables, una evolución de las herramientas de filtrado de paquetes, permite la inspección de estado de paquetes, mejorando la seguridad y eficiencia de los firewalls. Se configura para permitir tráfico de red interna y restringir acceso según reglas específicas.

**Configuración práctica:**  
Para que una red interna se conecte a Internet, se habilita el enrutamiento, se limpia la configuración anterior y se establece NAT para la red interna. También se pueden crear reglas específicas para limitar el tráfico a ciertos puertos y servicios.

#### 1.3 DHCP

**Asignación de direcciones IP:**  
El servidor DHCP automatiza la asignación de direcciones IP, lo cual es esencial para redes grandes o con dispositivos móviles. El servidor usa un archivo de configuración `/etc/dhcpd.conf` donde se definen parámetros como rangos de IP, servidores DNS y reservas de IP basadas en direcciones MAC.

**Configuración práctica:**  
Se instala y configura el servidor DHCP, se define un rango de IPs dinámico y se pueden hacer reservas específicas para ciertos dispositivos. El servidor DHCP asigna automáticamente direcciones IP y se inicia con el sistema.

Este resumen abarca la configuración de redes cableadas e inalámbricas, el uso de iptables para la gestión de tráfico y seguridad, y la implementación de DHCP para asignación dinámica de direcciones IP.
### 2. Compartir archivos e impresoras (Samba)

Samba es el método más utilizado para permitir la integración entre sistemas, ya que permite que los equipos Windows y GNU/Linux puedan compartir carpetas e impresoras entre sí. Samba es una colección de programas que hacen que Linux sea capaz de utilizar el protocolo SMB (Server Message Block), que es la base para compartir archivos e impresoras en una red Windows. Los posibles clientes para un servidor SMB incluyen Windows y otros sistemas GNU/Linux.

Samba está compuesto por tres paquetes: samba-common (archivos comunes), samba-client (cliente) y samba (servidor). Los paquetes que necesitas instalar dependen del uso que quieras darle al equipo.

Para instalar el cliente y servidor de Samba, es necesario ejecutar:
```bash
# apt-get install samba4 smbclient
```

A continuación, inicia el servicio ejecutando:
```bash
# service samba4 start
```

Para que Samba funcione correctamente, primero debes dar de alta a los usuarios del sistema y luego configurar los recursos a compartir.

#### 2.1. Gestión de usuarios
Samba realiza una gestión de usuarios independiente de la del sistema operativo. Por esta razón, necesitas dar de alta a los usuarios que vayan a utilizar Samba. El comando `smbpasswd` se utiliza para administrar los usuarios de Samba y sus contraseñas. La sintaxis del comando es:
```bash
# smbpasswd -opcion usuario
```

Donde `-opcion` es la opción a realizar y `usuario` es el nombre del usuario con el que quieres trabajar.

Por ejemplo, para añadir el usuario `juan`, debes ejecutar:
```bash
# smbpasswd -a juan
New SMB password:
Retype new SMB password:
Added user juan.
```

Y para eliminarlo:
```bash
# smbpasswd -x juan
Deleted user juan.
```

Para poder añadir un usuario en Samba, este tiene que existir en el sistema. Para dar de alta un usuario en el sistema, utiliza el comando `adduser`.

Para ver todos los usuarios de Samba, en las primeras versiones bastaba con ver el contenido del archivo `/etc/samba/smbpasswd`, pero en las versiones actuales los usuarios y contraseñas se guardan en la base de datos de Samba. Para ver los usuarios de Samba, ejecuta:
```bash
# pdbedit -w -L
```

#### 2.2. Compartir carpetas
Para compartir una carpeta, hay que modificar el archivo de configuración de Samba `/etc/samba/smb.conf`. El ejemplo más sencillo que se puede realizar es compartir una carpeta de forma pública para todos los usuarios. Para ello, añade:
```plaintext
[publico]
path = /publico
public = yes
read only = yes
```

**Opciones más utilizadas de smb.conf:**

| Opción        | Comentario                                                                                     |
|---------------|------------------------------------------------------------------------------------------------|
| `[recurso]`   | Nombre del recurso compartido.                                                                 |
| `browseable`  | Indica si se puede explorar dentro del recurso. Valores: `no` y `yes`.                         |
| `comment`     | Información adicional sobre el recurso (no afecta su operación).                               |
| `create mode` | Especifica los permisos por defecto de los archivos creados.                                    |
| `directory mode` | Especifica los permisos por defecto de los directorios creados.                              |
| `force user`  | Especifica el usuario propietario de los archivos y carpetas que se crean.                     |
| `force group` | Especifica el grupo propietario de los archivos y carpetas que se crean.                       |
| `guest ok`    | Indica si se permite el acceso a usuarios anónimos. Valores: `no` y `yes`.                     |
| `path`        | Carpeta a compartir.                                                                           |
| `public`      | Indica si el directorio permite el acceso público. Valores: `no` y `yes`.                      |
| `read only`   | Indica que el directorio es solo lectura. Valores: `no` y `yes`.                               |
| `valid users` | Indica los usuarios que pueden acceder a la carpeta.                                           |
| `writable`    | Indica que se puede modificar el contenido de la carpeta.                                       |
| `write list`  | Indica los usuarios que pueden modificar el contenido.                                          |

Para establecer que el recurso sea accesible solo por determinados usuarios:
```plaintext
[miñascousas]
path = /datos/
comment = “Datos e aplicacións”
valid users = juan, encarni, @master
```

Lógicamente, los usuarios se deben haber creado previamente y el grupo `master` debe existir en el archivo `/etc/group`.

Para establecer permisos de escritura para el usuario `juan` y permisos de lectura para el usuario `encarni` y el grupo `master`:
```plaintext
[miñascousas]
path = /datos/
comment = “Datos e aplicacións”
valid users = juan, encarni, @master
writable = yes
write list = juan
read list = juan, @master
force user = juan
force group = juan
create mode = 770
directory mode = 770
```

Después de compartir una carpeta, es necesario establecer los permisos en el archivo de configuración y en el sistema de archivos utilizando los comandos `chmod`, `chown` y `chgrp`. Finalmente, para que se apliquen los cambios, reinicia el servicio:
```bash
# service samba4 restart
```

#### 2.3. Compartir impresoras
Existen dos formas de compartir las impresoras conectadas al equipo para que las utilicen todos los clientes de la red: a través de la herramienta gráfica `system-config-printer` o utilizando Samba.

Para compartir una impresora, añade en el archivo de configuración de Samba `/etc/samba/smb.conf` un nuevo recurso:
```plaintext
[printers]
comment = All printers
path = /var/spool/samba
browseable = no
printable = yes
public = no
writable = no
create mode = 0700
```

El acceso a las impresoras GNU/Linux desde Windows funciona de la misma forma que los directorios. El nombre compartido es el nombre de la impresora Linux en el archivo `printtab`. Por ejemplo, para acceder a la impresora `HP_laserjet`, los usuarios de Windows deberán acceder a `\\smbserv\HP_laserjet`.

**Opciones más utilizadas de smb.conf (sección printers):**

| Parámetro    | Comentario                                                                                         |
|--------------|----------------------------------------------------------------------------------------------------|
| `comment`    | Proporciona información sobre la sección (no afecta su operación).                                 |
| `path`       | Especifica la ruta de acceso a la cola de impresión o spool (por defecto `/var/spool/samba`).       |
| `browseable` | Indica si se puede ver la impresora. Valores: `no` y `yes`.                                         |
| `printable`  | Debe ponerse `yes` para que funcionen las impresoras.                                               |
| `public`     | Indica si cualquier usuario puede imprimir. Valores: `no` y `yes`.                                  |
| `writable`   | Las impresoras no son escribibles, por lo tanto, debe ser `no`.                                     |

#### 2.4. Asistentes de configuración
Dado el gran uso de Samba para compartir información entre sistemas Windows y GNU/Linux, existen varias interfaces que facilitan el proceso de configuración del sistema. Las interfaces más importantes son:

- **Swat:** Es una interfaz web específica para administrar Samba.
  ```bash
  # apt-get install swat
  ```
  Para acceder, abre un navegador y escribe `http://127.0.0.1:901`.

- **Webmin:** Permite configurar cualquier servicio del servidor. Para acceder al módulo de configuración, pulsa en Servers > Samba Windows File Sharing.

- **system-config-samba:** Herramienta de XWindows para administrar Samba.
  ```bash
  # apt-get install system-config-samba
  ```
  Además, instala las siguientes dependencias:
  ```bash
  # apt-get install gksu python-gtk2 python-glade2
  ```

#### 2.5. Cliente
Además de actuar como servidor de archivos, el equipo puede utilizarse como cliente para acceder a los recursos compartidos en otros servidores. Existen varias formas para acceder desde GNU/Linux a carpetas e impresoras compartidas. La forma más sencilla es mediante los programas cliente que vienen con la instalación de Samba: `smbclient` y `smbprint`. Sin embargo, esta solución está algo limitada, particularmente en el acceso a archivos. `smbclient` proporciona una forma similar a un servidor FTP para acceder a un recurso remoto compartido, pero no permite usar comandos normales de Unix como `cp` y `mv`.

Para montar un recurso compartido de Samba, primero crea la carpeta donde se va a montar el recurso y luego ejecuta el comando `mount`:
```bash
$ mkdir /prueba
$ mount -t cifs -o user=usuario,pass=contrasena //10.0.0.1/recurso /prueba
```

Donde:
- `-t cifs`: Indica el tipo de archivos que se va a utilizar (en este caso, CIFS).
- `-o user=usuario,pass=contrasena`: Especifica el nombre del usuario y la contraseña para acceder.
- `//10.0.0.1/recurso`: Dirección IP y nombre del recurso al que quieres acceder.
- `/prueba`: Directorio donde se va a montar el recurso compartido.

Para montar el

 recurso automáticamente al iniciar el equipo, añade al archivo `/etc/fstab` la siguiente línea:
```plaintext
//10.0.0.1/recurso /prueba cifs rw,username=login,password=pass 0 0
```

**Datos más importantes del servicio Samba:**

| Parámetro               | Detalle                                      |
|-------------------------|----------------------------------------------|
| Nombre del servicio     | samba4                                       |
| Archivo de configuración| `/etc/samba/smb.conf`                        |
| Comandos más utilizados | `smbpasswd`, `smbclient`, `pdbedit`, `mount` |
| Puertos utilizados      | 137/UDP, 138/UDP, 139/TCP y 445/TCP          |
### 3. NFS (Network File System)

NFS é un servizo que permite que os equipos GNU/Linux compartan cartafoles entre si. Baséase no modelo cliente/servidor, de forma que un servidor comparte un cartafol para que os clientes poidan utilizalo. Unha vez que un cliente monta un cartafol compartido, pode utilizalo coma se tratásese dun cartafol do sistema de ficheiros local.

Para instalar o servizo NFS, debes executar:
```bash
# apt-get install nfs-kernel-server nfs-common portmap
```

Antes de iniciar a configuración, hai que iniciar o servizo executando:
```bash
# service nfs-kernel-server start
```

#### 3.1. Compartir unha carpeta
Para indicar os directorios que se desexan compartir, hai que modificar o ficheiro `/etc/exports`. A sintaxe é:
```plaintext
<directorio> <IP>(permisos) <IP>(permisos)...
```

Os permisos que se poden establecer son: `rw` (lectura e escritura) e `ro` (lectura). Por exemplo, para compartir o cartafol `/datos` de forma que o equipo `192.168.20.9` poida acceder en modo lectura e escritura, e o equipo `192.168.20.8` tan só poida acceder en modo lectura, escríbese:
```plaintext
/datos 192.168.20.9(rw) 192.168.20.8(ro)
```

O cartafol compártese soamente á IP establecida no ficheiro `/etc/exports` polo usuario `nfsnobody`. Para establecer os permisos adecuados, executa:
```bash
# chmod 660 /datos -R
# chown nfsnobody /datos -R
# chgrp nfsnobody /datos -R
```

Como o usuario `nfsnobody` ten un UID e GID diferente en cada equipo, é recomendable asignarlle o mesmo identificador modificando os ficheiros `/etc/passwd` e `/etc/groups` tanto nos equipos clientes como servidores.

Unha vez compartida a carpeta, reinicia o servizo executando:
```bash
# service nfs-kernel-server restart
```

#### 3.2. Configuración do cliente
Para acceder ao directorio que comparte o servidor, hai que montalo, xa sexa manualmente ou automaticamente ao iniciar o equipo. Para montar o sistema de ficheiros no cliente, executa:
```bash
# mount 192.168.20.100:/datos /proba
```

Onde:
- `192.168.20.100:/datos` é a carpeta que se compartiu no servidor no ficheiro `/etc/exports`.
- `/proba` é a carpeta onde se monta a carpeta compartida.

Se desexas montar a carpeta automaticamente ao iniciar o sistema, hai que modificar o ficheiro `/etc/fstab` engadindo a seguinte liña:
```plaintext
192.168.20.100:/datos  /proba  nfs  rw,hard,intr  0 0
```

Onde:
- `rw`: Indica que se monta o directorio en modo lectura/escritura. Para montalo só en modo lectura, utiliza `ro`.
- `hard`: Indica que se ao copiar un ficheiro na carpeta compartida se perde a conexión co servidor, se volverá a iniciar a copia do ficheiro cando o servidor se encontre activo.
- `intr`: Evita que as aplicacións queden "colgadas" ao intentar escribir na carpeta se non se encontra activa.

#### Datos máis importantes do servizo NFS
- **Nome do servizo:** `nfs`
- **Carpetas compartidas:** `/etc/exports`
- **Comandos máis utilizados:** `mount`
- **Portos:** `2049/TCP` e `2049/UDP`

Coa configuración descrita, podes compartir carpetas entre equipos GNU/Linux utilizando o servizo NFS, garantindo o acceso controlado mediante permisos e facilitando a integración de recursos en rede.
### 4. Acceso remoto ao sistema

Os servizos máis utilizados para acceder de forma remota a un sistema GNU/Linux son:

- **Telnet:** Permite acceder ao sistema de forma remota dun xeito non seguro.
- **Open SSH:** Permite acceder ao sistema por terminal de forma segura, cifrando as comunicacións.
- **VNC:** Permite utilizar o servidor utilizando o escritorio instalado no sistema (GNOME ou KDE).

#### 4.1. SSH

SSH é un protocolo que permite conectarse de forma segura a un servidor para administralo, realizar transmisión de ficheiros, utilizar FTP seguro, e incluso como transporte doutros servizos. O protocolo SSH garante que a conexión realízase desde os equipos desexados usando certificados e establece unha comunicación cifrada mediante un algoritmo robusto, normalmente de 128 bits.

##### Instalación e configuración de OpenSSH

OpenSSH instálase por defecto en moitas distribucións GNU/Linux. Se non está instalado, podes facelo executando:
```bash
# apt-get install ssh
```
Para iniciar o servizo, executa:
```bash
# service ssh start
```
Para que o servizo execútese automaticamente ao iniciar o sistema, executa:
```bash
# chkconfig ssh on
```

Para protexer o sistema contra ataques de forza bruta, é recomendable usar **fail2ban**. Fail2ban bloqueará a IP despois de 5 intentos errados de autentificación.
Máis información en [fail2ban.org](http://www.fail2ban.org/).

##### Configuración de OpenSSH

O ficheiro de configuración principal é `/etc/ssh/sshd_config`. Algúns dos parámetros máis importantes son:

- **Port e ListenAddress:** Permiten cambiar o porto e a dirección nas que o servidor SSH atenderá peticións:
    ```plaintext
    Port 22
    ListenAddress 0.0.0.0
    ```

- **PermitRootLogin:** Permite ou non o acceso do usuario root:
    ```plaintext
    PermitRootLogin no
    ```

- **AllowUsers:** Permite restrinxir o acceso a determinados usuarios:
    ```plaintext
    AllowUsers cesar sonia
    ```

  Para restrinxir tamén por equipo anfitrión:
    ```plaintext
    AllowUsers cesar@10.0.0.2 sonia@10.0.0.2
    ```

- **PrintMotd e Banner:** Para mensaxes de entrada e conexión:
    ```plaintext
    PrintMotd yes
    Banner /etc/issue.net
    ```

- **Configuración de seguridade e control de acceso:**
    ```plaintext
    IgnoreUserKnownHosts no
    GatewayPorts no
    AllowTcpForwarding yes
    ```

- **Uso de subsistemas para outras aplicacións, como FTP:**
    ```plaintext
    Subsystem sftp /usr/lib/openssh/sftp-server
    ```

Para aplicar os cambios, reinicia o servizo:
```bash
# /etc/init.d/ssh restart
```

##### Cliente SSH

Para conectarte a un servidor desde un equipo Linux, executa:
```bash
$ ssh <equipo>
```
Onde `<equipo>` pode ser o nome do equipo ou a dirección IP.

Para conectar desde Windows, utiliza a aplicación **PuTTY**: [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/).

Para copiar ficheiros a través de SSH, usa o comando `scp`:
```bash
$ scp /etc/passwd 10.0.0.2:/root
```

Para permitir a utilización dos comandos SSH e SCP sen necesidade de escribir o contrasinal, consulta [esta guía](http://www.adminso.es/wiki/index.php/Ssh_sin_contraseña).

##### Datos importantes do servizo SSH

- **Nome do servizo:** `sshd`
- **Ficheiro de configuración:** `/etc/ssh/sshd_config`
- **Hosts permitidos:** `/etc/host.allow`
- **Equipos autorizados para acceder por SSH sen contrasinal:** `$HOME/.ssh/authorized_keys`
- **Comandos máis utilizados:** `ssh`, `scp`, `sftp`
- **Porto utilizado:** `22/TCP`

#### 4.2. VNC

VNC é un programa con licenza GPL que permite acceder a un equipo remoto utilizando a súa contorna gráfica.

##### Instalación do servidor VNC

Para instalar o servidor VNC, executa:
```bash
# apt-get install tightvncserver
```

Indica o contrasinal do servidor VNC:
```bash
# vncpasswd
```

Para crear automaticamente os ficheiros de configuración e iniciar o servizo, executa:
```bash
# vncserver
```

##### Datos importantes do servizo VNC

- **Nome do servizo:** `vncserver`
- **Ficheiro de configuración:** `/etc/sysconfig/vncservers`
- **Comandos máis importantes:** `vncpasswd`, `vncserver`
- **Portos:** `6000/tcp`, `6001/tcp`, `6002/tcp`, `6003/tcp`

##### Cliente VNC

Para acceder ao servidor, podes utilizar calquera cliente VNC. En sistemas GNU/Linux, utiliza **Vinagre**:
```bash
# apt-get install vinagre
```

Accede ao menú Aplicacións, Internet, e executa a aplicación **Remote Desktop Viewer**. Pulsa **Connect**, indica a dirección do servidor VNC (por exemplo, `10.0.0.1:5901`), e pulsa **Connect** para acceder.

En sistemas Windows, utiliza **tightVNC**:
1. Descarga tightVNC: [tightVNC](http://www.tightvnc.com/)
2. Instala no equipo o visor tightVNC.
3. Executa **tightVNC Viewer**.
4. Indica a dirección IP do servidor e o porto (por exemplo, `10.0.0.1:5901`).
5. Pulsa **Connect**, introduce o contrasinal do servidor VNC e xa tes acceso ao escritorio do servidor.

Os servizos SSH e VNC permiten acceder remotamente ao sistema GNU/Linux, cada un ofrecendo diferentes niveis de interacción e seguridade, adaptándose ás necesidades específicas de administración remota.
### 5. Servidor Web

Para instalar o servidor Apache desde os repositorios, executa o seguinte comando:
```bash
# apt-get install apache2
```

O servizo iníciase automaticamente ao arranque do sistema:
```bash
# chkconfig apache2 on
```

Para iniciar o servizo agora mesmo:
```bash
# service apache2 start
```

Unha vez instalado, Apache publica automaticamente o contido do directorio `/var/www`. Así, para publicar unha páxina web, crea o contido en dito directorio. Para acceder á páxina principal do servidor, escribe na barra de direccións do teu navegador `http://localhost/` ou `http://dirección_ip/`.

#### 5.1. Instalar módulo PHP

PHP é unha linguaxe de programación interpretada polo servidor de páxinas web para xerar contido dinámico. Pódese utilizar tamén desde unha interface de liña de comandos ou para a creación de aplicacións gráficas. Para máis información sobre PHP, visita [php.net](http://php.net/).

Para instalar PHP automaticamente, executa:
```bash
# apt-get install php5
```

Para comprobar que PHP instalouse con éxito, crea un ficheiro PHP no directorio raíz do servidor web. Por exemplo, para mostrar información sobre a instalación actual de PHP, edita o ficheiro `/var/www/info.php`:
```bash
# nano /var/www/info.php
```

Inclúe o seguinte contido no ficheiro:
```php
<?php
phpinfo();
?>
```

Antes de probar este ficheiro, reinicia o servidor Apache:
```bash
# service apache2 restart
```

Agora, inicia un navegador web e escribe na barra de direccións `http://localhost/info.php`. Se ves unha páxina con información sobre PHP, entón PHP está correctamente instalado.

#### 5.2. Configuración

A configuración de Apache almacénase no directorio `/etc/apache2`. As opcións de configuración máis utilizadas están en varios ficheiros:

- **/etc/apache2/ports.conf:** Permite establecer os portos de escoita para as comunicacións HTTP normais (porto 80) e as comunicacións seguras HTTPS (porto 443).
    ```plaintext
    Listen *:80
    Listen *:443
    ```

- **/etc/apache2/apache2.conf:** Permite establecer o usuario e grupo aos que pertencen os procesos que executa o servidor.
    ```plaintext
    User www-data
    Group www-data
    ```

Apache almacena a configuración de cada sitio web no directorio `/etc/apache2/sites-available`. Por defecto, están os sitios `default` e `default-ssl`. Cada sitio ten a seguinte estrutura:
```plaintext
<virtualhost *:80>
    ServerAdmin servermaster@localhost
    # Servername www.miempresa.com # comentado en default
    DocumentRoot /var/www
    DirectoryIndex index.html default.html
</virtualhost>
```

- **ServerAdmin:** O correo electrónico do administrador do sitio web.
- **ServerName:** O nome FQDN do sitio web. Para o dominio `default` non se indica ningún nome, pero para atender peticións específicas de dominios (por exemplo, `www.miempresa.com`) si se debe establecer.
- **DocumentRoot:** Indica a localización onde se encontran as páxinas web do sitio.
- **DirectoryIndex:** Indica o nome dos ficheiros que se envían por defecto.

##### Novo sitio

Para engadir un dominio `www.miempresa.com` que se aloxa na carpeta `/portales/miempresa`, crea o ficheiro `/etc/apache2/sites-available/miempresa.com` co seguinte contido:
```plaintext
<virtualhost *:80>
    ServerName www.miempresa.com
    DocumentRoot /portales/miempresa
</virtualhost>
```

Para activar o sitio:
```bash
# a2ensite miempresa.com
```

Reinicia o servidor web:
```bash
# service apache2 restart
```

Loxicamente, para que o servidor web atenda a un determinado dominio, a entrada DNS (por exemplo, `www.miempresa.com`) debe apuntar ao servidor web.

##### Sitio seguro (HTTPS)

Para utilizar HTTPS, realiza os seguintes pasos:

1. Activa o módulo SSL:
    ```bash
    # a2enmod ssl
    ```

2. Activa o sitio `default-ssl` ou crea un novo sitio web:
    ```bash
    # a2ensite default-ssl
    ```

3. Reinicia o servidor web:
    ```bash
    # service apache2 restart
    ```

Unha vez finalizado o proceso, accede a un navegador web e escribe `https://IP_Servidor`.

Podes xerar o teu propio certificado de seguridade utilizando o comando `openssl`.

Para aprender máis sobre Apache, consulta [adminso.es](http://www.adminso.es/index.php/Apache).

Para iniciar e parar o servidor web, utiliza o comando `service`. Por exemplo, para iniciar o servizo, executa:
```bash
# service apache2 start
```

Tamén podes parar o servizo (`stop`), reinicialo (`restart`), ou recargar a configuración (`reload`).

##### Datos importantes do servidor Apache

- **Nome do servizo:** `apache2` (Ubuntu)
- **Ficheiro de configuración:** `/etc/httpd/conf/httpd.conf`
- **Directorio web:** `/var/www` (Ubuntu)
- **Comandos máis utilizados:** `htpasswd`
- **Portos:** `80/tcp` e `443/tcp`

### 6. Servidor FTP

Vsftpd (Very Secure FTP) é un servidor FTP moi pequeno e seguro. Para instalar o servidor FTP no sistema, instala o paquete `vsftpd`. Podes facelo a través da liña de comandos ou Synaptic:
```bash
# apt install vsftpd
```

Unha vez instalado o paquete, inicia o servizo executando:
```bash
# service vsftpd start
```

Para comprobar que o servidor está funcionando correctamente, conéctate ao servidor:
```bash
$ ftp localhost
Connected to localhost (127.0.0.1).
220 (vsFTPd 2.3.0)
Name (localhost:root): usuario
331 Please specify the password.
Password:
230 Login successful. Have fun.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV
150 Here comes the directory listing.
-rw-r--r-- 1 1003 1003 179 Mar 15 18:00 examples.desktop
226 Directory send OK.
ftp> quit
221 Goodbye.
```

Se o servidor está correctamente instalado pero non permite o acceso desde o exterior, pode ser que o router non estea configurado para deixar pasar o tráfico do servidor FTP. Para aprender a configurar e protexer o servidor vsftpd, consulta [adminso.es](http://www.adminso.es/index.php/Vsftpd).

Nunca configures o servidor FTP para permitir o acceso anónimo nin permitas a escritura sen enxaular os usuarios do sistema.

##### Datos importantes do servidor VSFTP

- **Nome do servizo:** `vsftpd`
- **Ficheiro de configuración:** `/etc/vsftpd.conf`
- **Porto utilizado:** `21/tcp`
