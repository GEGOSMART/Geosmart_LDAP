INSTRUCCIONES DE EJECUCIÓN

Para ejecutar en un nodo que tenga Ubuntu, es necesario tener docker-compose y ejecutar el siguiente comando en la carpeta donde esté el docker-compose.yml: 

sudo docker-compose up

Esto desplegará el servidor LDAP en el puerto 389, y el cliente web para LDAP phpldapadmin, en el puerto 8085

COMANDOS LDAP-UTILS (UBUNTU)

-> Para instalar el package de las herramientas OpenLDAP para la conexión remota por consola de Ubuntu:

sudo apt-get update
sudo apt-get install ldap-utils

-> Todos los resultados pueden ser consultados en el cliente LDAP del nodo, en la dirección http://18.210.193.21:8085

-> Consultar todos los usuarios de la org unit "development"

ldapsearch -H ldap://18.210.193.21 -D "cn=admin,dc=swarch,dc=geosmart,dc=com" -w "admin" -b "ou=development,dc=swarch,dc=geosmart,dc=com" -s sub

-> Consultar un usuario a partir de su "uid"

ldapsearch -H ldap://18.210.193.21 -D "cn=admin,dc=swarch,dc=geosmart,dc=com" -w "admin" -b "uid=miapenahu,ou=development,dc=swarch,dc=geosmart,dc=com"

-> Crear un usuario en la Organisational Unit

Nota: Para esto es conveniente ir a la carpeta "ldif", para no especificar más que el nombre del archivo .ldif que se va a usar 

ldapadd -H ldap://18.210.193.21 -D "cn=admin,dc=swarch,dc=geosmart,dc=com" -w "admin" -f create-user-miapenahu.ldif

-> El archivo create-user-miapenahu.ldif contiene la siguiente información:

dn: uid=miapenahu,ou=development,dc=swarch,dc=geosmart,dc=com
objectClass: top
objectclass: inetOrgPerson
objectClass: posixAccount
gn:Miguel Alejandro
sn:Peña Hurtado
cn: miapenahu@unal.edu.co
uid: miapenahu
uidNumber: 1000
gidNumber: 500
homeDirectory: /home/miapenahu
loginShell: /bin/bash
userPassword: {crypt}x

Nota1: Por razones de seguridad la contraseña no se especifica aquí, debido a que quedaría guardada en texto plano

Nota2: gidNumber es 500 porque ese número identifica a los usuarios dentro del sistema

-> Luego, hay que cambiarle la contraseña para que se almacene encriptada de manera correcta

-> Actualizar contraseña para un usuario

ldappasswd -H ldap://18.210.193.21 -D "cn=admin,dc=swarch,dc=geosmart,dc=com" -w "admin" "uid=miapenahu,ou=development,dc=swarch,dc=geosmart,dc=com" -s Abcd1234 

-> Modificar el uid (y ubicación si es necesario) para un usuario

ldapmodify -H ldap://18.210.193.21 -D "cn=admin,dc=swarch,dc=geosmart,dc=com" -w "admin" -f mod-uid-miapenahu.ldif

-> El archivo mod-uid-miapenahu.ldif contiene la siguiente información:

dn: uid=miapenahu,ou=development,dc=swarch,dc=geosmart,dc=com
changetype: moddn
newrdn: uid=miapenahu2
deleteoldrdn: 1
newsuperior: ou=development,dc=swarch,dc=geosmart,dc=com

Nota: Si se quiere cambiar de org unit, hay que poner el nombre en ou en newsuperior (La org unit de destino ya debe estar creada!!!)

-> Modificar los datos de un usuario (excepto el RDN):

ldapmodify -H ldap://18.210.193.21 -D "cn=admin,dc=swarch,dc=geosmart,dc=com" -w "admin" -f mod-data-miapenahu2.ldif

Nota: Se asume que ya se le cambió el uid de "miapenahu" a "miapenahu2"

-> El archivo mod-data-miapenahu2 tiene la siguiente información:

dn: uid=miapenahu2,ou=development,dc=swarch,dc=geosmart,dc=com
changetype: modify
replace: gn
gn: Firstname
-
replace: sn
sn: Lastname
-
replace: cn
cn: miapenahu@example.com
-
replace: uidNumber
uidNumber: 999

-> Eliminar usuario de la base de datos

ldapdelete -H ldap://18.210.193.21 -D "cn=admin,dc=swarch,dc=geosmart,dc=com" -w "admin" "uid=miapenahu2,ou=development,dc=swarch,dc=geosmart,dc=com"

-> Autenticar un usuario en la base de datos

-> La contraseña no es correcta

ldapwhoami -H ldap://18.210.193.21 -D "uid=miapenahu,ou=development,dc=swarch,dc=geosmart,dc=com" -w "@not-the-password"

-> Esto responde:

ldap_bind: Invalid credentials (49)

-> La contraseña es correcta (se asume que la contraseña fue cambiada con el comando de arriba)

ldapwhoami -H ldap://18.210.193.21 -D "uid=miapenahu,ou=development,dc=swarch,dc=geosmart,dc=com" -w "Abcd1234"

-> Esto responde con la ruta completa del usuario:

dn:uid=miapenahu,ou=development,dc=swarch,dc=geosmart,dc=com
