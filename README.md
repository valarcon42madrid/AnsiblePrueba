# Parte I – Gestión de usuarios y variables
* Inventario

Se creó un archivo inventory con el grupo [servidores] que contiene vagrant1 y vagrant2.

* Archivo de configuración

Se creó ansible.cfg apuntando al inventario:


[defaults]
inventory = inventory

* Playbook users.yml

Se define un play que:

Se ejecuta sobre el grupo servidores

Crea los usuarios carlos y marta

Define la variable packages con memcached y mariadb-server

Instala redis solo si el host tiene más de 20 MB de swap:

when: ansible_swaptotal_mb > 20

* Verificación: verify_user.yml

Se corrige el uso incorrecto del módulo user con --check.

Se implementa una verificación con command: id carlos y un bloque debug.

Se escribe un resultado a verify.txt si carlos existe.

# ✅ Parte II – Despliegue web y validación

* Playbook dev_deploy.yml

Se encarga de:

Instalar Apache (httpd)

Iniciar y habilitar el servicio

Desplegar un vhost.conf desde una plantilla Jinja2

Crear el directorio /var/www/vhosts/{{ ansible_hostname }}

Copiar un archivo HTML estático (index.html)

Habilitar los puertos HTTP/HTTPS en firewalld

Incluir un handler para reiniciar Apache

* Plantilla templates/vhost.conf.j2

Define un VirtualHost apuntando al directorio específico por hostname.

* Archivo files/index.html

Contiene un HTML estático con un mensaje genérico:

Hola desde el servidor vagrant

* Playbook get_web_content.yml

Se ejecuta en el nodo de control.

Usa un bloque block/rescue para:

Intentar recuperar contenido de http://vagrant1

Si falla, escribir los detalles en error.log

* Playbook maestro site.yml

Importa y ejecuta ambos playbooks:

- import_playbook: dev_deploy.yml
- import_playbook: get_web_content.yml
  
🔎 Comprobaciones finales


** Puedes verificar que todo funciona correctamente con los siguientes comandos:


1. 🔥 Apache

systemctl status httpd

Debe estar activo (running) y habilitado.

2. 🧾 Virtual host

cat /etc/httpd/conf.d/vhost.conf

Debe contener una configuración basada en {{ ansible_hostname }}.

3. 📁 Archivos en el vhost

ls /var/www/vhosts/$(hostname -s)
cat /var/www/vhosts/$(hostname -s)/index.html

Debe mostrar el archivo index.html copiado desde files/.

4. 🌐 Acceso web desde el nodo de control

curl http://vagrant1

Debe devolver el contenido HTML con el mensaje: Hola desde mi servidor web.

5. 🔐 Firewall

firewall-cmd --list-services

Debe listar al menos: http https.

6. 📄 Error log de verificación web
   
Si la verificación web falló, revisa:

cat error.log
