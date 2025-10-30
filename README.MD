Autor: Ángel Nogueira Castaño

# Descripción del proyecto

> El objetivo de esta práctica es instalar y configurar Apache en una máquina virtual (Ubuntu 22.04) y desplegar
un sitio web estático sencillo con navegación entre páginas, imágenes y estilos.

# Pasos seguidos para la instalación y despliegue.
1. Creación de la VM

    Crea una VM como clon enlazado de la VM base Ubuntu-Server-Base.ova proporcionada en el repositorio del curso:
    
    VBoxManage clonevm "Ubuntu-Server-Base" --name "Ubuntu-Apache-A2-1" --snapshot "After-Config-Proxy" --options link --register

    Cambia el nombre del snapshot si has usado otro nombre al crear la VM base tras instalar el script de configuración inicial.

    Configura la red: Adaptador 1 en NAT + Port-Forward (host 8081 → guest 80; host 2223 → guest 22):
    VBoxManage modifyvm "Ubuntu-Apache-A2-1" --natpf1 delete "ssh"

    VBoxManage modifyvm "Ubuntu-Apache-A2-1" --natpf1 delete "http"

    VBoxManage modifyvm "Ubuntu-Apache-A2-1" --natpf1 delete "https"

    # Crear nuevas reglas con puertos diferentes
    VBoxManage modifyvm "Ubuntu-Apache-A2-1" --natpf1 "ssh,tcp,,2223,,22"

    VBoxManage modifyvm "Ubuntu-Apache-A2-1" --natpf1 "http,tcp,,8081,,80"

    VBoxManage modifyvm "Ubuntu-Apache-A2-1" --natpf1 "https,tcp,,8444,,443"

    Inicia la VM y accede por SSH desde el host:ssh -p 2223 dawserver@localhost

2. Preparación del entorno

    Actualiza paquetes:
    sudo apt update && sudo apt upgrade -y

    Instala apache2:
    sudo apt install -y apache2

    Configura el firewall para permitir tráfico HTTP y HTTPS:
    sudo ufw allow 'Apache Full'
    Verifica que las reglas se han aplicado:
    sudo ufw status

    Si no está configurado, habilita el tráfico ssh, HTTP y HTTPS:
    sudo ufw allow ssh
    sudo ufw allow 80
    sudo ufw allow 443
    sudo ufw reload

    Habilita el firewall si no está habilitado:
    sudo ufw enable

3. Comprobación básica

Comprueba que el servicio está activo:
sudo systemctl status apache2Desde el host, accede a http://localhost:8081/ y comprueba que ves la página por defecto
de Apache. Haz una captura de pantalla.

4. Desplegar el sitio de ejemplo

    Copia los archivos del sitio de ejemplo a /var/www/html/. 
    El sitio de ejemplo incluye:
    index.html — Página principal con bienvenida y navegación.
    about.html — Página interna con información sobre el proyecto.
    contact.html — Página con formulario de contacto (estático) y datos.
    css/style.css — Estilos básicos.
    img/ — Carpeta con imágenes (logo y dos imágenes de contenido).

    Utiliza scp desde tu máquina local:
    scp -P 2223 -r /ruta/local/del/sitio/*
    dawserver@localhost:~/practicas/apache #Copiar desde local al home 

    Luego, en la VM:
    # 1. Crear el directorio raíz del sitio web (si no existe)
    sudo mkdir -p /var/www/html
    # 2. Limpiar contenido previo (OPCIONAL - elimina todo lo que haya)
    sudo rm -rf /var/www/html/*
    Comprueba que si recargas http://localhost:8081/ ves una página en blanco (o error 404).
    Ahora, copia los archivos:
    # 3. Copiar los archivos del sitio recursivamente desde el home al
    directorio web
    sudo cp -r ~/practicas/apache/ejemplo_sitio_apache/* /var/www/html/

    Vuelve a recargar http://localhost:8081/ y comprueba que ves la página principal del sitio de ejemplo. ¿Se cargan las imágenes y estilos? Navega entre las páginas. Abre las herramientas de desarrollo del navegador (F12) y comprueba las peticiones de red (pestaña Network). Recarga la página con las herramientas abiertas y verifica los códigos de estado de cada uno de los recursos solicitados (HTML, CSS, imágenes). Haz captura de pantalla. 
    
    Ahora, ajusta los permisos y propiedad de los archivos para que Apache pueda servirlos correctamente:
    
    # 4. Cambiar el propietario de todos los archivos, de manera recursiva, al usuario y grupo del servidor web (www-data)  
    sudo chown -R www-data:www-data /var/www/html
    
    # 5. Establecer permisos de lectura/ejecución para todos 755: propietario (www-data) puede leer, escribir y ejecutar; grupo y otros pueden leer y ejecutar 
    sudo chmod -R 755 /var/www/html/
    # 6. Limpiar carpeta temporal del home (opcional)
    sudo rm -rf ~/practicas
    # 7. Recargar la configuración de Apache
    sudo systemctl reload apache2

5. Comprobar
Accede a la página principal desde el host y verifica navegación, imágenes y estilos. Ingresa en el
navegador: http://localhost:8081/

6. Apaga la VM y crea un snapshot llamado After-Apache-Deployment para posibles futuras prácticas.

# Capturas de pantalla del proceso: configuración, comprobaciones, estructura de archivos, página cargada y navegación.

Crea una VM como clon enlazado de la VM base Ubuntu-Server-Base.ova proporcionada en
el repositorio del curso:

![clon_enlazado](/despr2_1_apache/img/virtualbox%20clon%20creada.png)

Configura la red: Adaptador 1 en NAT + Port-Forward (host 8081 → guest 80; host 2223 → guest
22):

![config_red](img/configuracion%20de%20red.png)

Instalar apache2:
![installapache2](img/instalr%20apache2.png)


Status de apache2:
![statusapache2](img/status%20apache2.png)

Acceder a la pagina por defecto:
![defaultpage](img/local%20base.png)

Acceder a la pagina con los estilos creados:
![stylepage](img/local%20mod.png)

Crea un snapshot llamado After-Apache-Deployment:
![snapshotafterapache](img/crear%20snapshot%20luego%20de%20instalar%20apache.png)

Estructura de los archivos creado:
![Estructurapermiso](img/directorios%20con%20permisos.png)

![Estructuralistado](img/structura%20de%20directorios%20sin%20mostrar%20permisos.png)


# Problemas encontrados durante la configuración y despliegue, y cómo los solucionaste.
Algun problema al hacer alguna snapshot con los comandos solución hacerlo manualmente.

