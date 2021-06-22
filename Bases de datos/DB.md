# Administración de servidores de bases de datos

# Introducción
Elegir un motor de base de datos a utilizar entre todas las opciones que tenemos disponibles en la actualidad no es una tarea fácil. Por supuesto, esto puede ser solo el inicio porque luego de elegir el motor de DB, necesitamos aprender cómo desplegarlo e interactuar con el.


## Instanciar un servidor de base de datos
Lo primero que debemos hacer para administrar un servidor de base de datos es deplegarlo. En aspectos generales esto implica los siguientes pasos:

### Provisionar el servidor
Dependiendo del nivel de administración que tengamos, esto puede significar desde desplegar un servidor físico completo (algo poco común en la actualidad), desplegar una máquina virtual en un servidor de virtualización, crear una máquina virtual en un proveedor de nube, hasta crear una instancia administrada en una nube. Los pasos o procedimientos para cada uno de los escenarios presentados puede variar, pero el resultado sigue siendo el mismo, tener un servidor de base datos a disposición para que pueda prestar un servicio. En este caso particular, almacenar información para un requerimiento de negocio específico.
En el ambiente de prueba/aprendizaje que nosotros estamos trabajando, desplegamos el servidor de base de datos en un contenedor. Esto nos permite hacer el despliegue en cuestión de minutos. Podemos hacerlo de dos maneras:
- Descargar la imagen e instanciar el contenedor
- Instanciar el contenedor (si la imagen no existe en el repositorio local entonces la descargará automáticamente)

Aunque podemos utilizar imágenes de sistemas operativos como CentOS, Ubuntu, Debian o Fedora y luego hacer la instalación de la plataforma de base de datos, los fabricantes de estas plataformas han creado imágenes personalizadas con archivos de configuración precargados de modo que podamos enfocarnos en el producto y no el proceso. Si te interesa saber cómo hacer instalación manual de alguno de estos, puedes hacer búsquedas simples con el formato "Instalación de *plataforma* en *sistema operativo*" reemplazando plataforma y sistema operativo, respectivamente. 

Para hacerlo de la primera forma entonces ejecutamos el comando siguiente:
`docker pull mysql`
El comando descargará la imagen `mysql` utilizando la etiqueta `latest`, al no haberla especificado en la línea de comandos.

Para la segunda manera utilizamos el comando:
`docker run -d --name db01 --hostname db01 --env MYSQL_ROOT_PASSWORD='contraseña' mysql`

donde:
- `-d`: Indica que el contenedor se ejecutará como un servicio en segundo plano, modo background
- `--name`: El nombre del contendor para el sistema anfitrión, es decir, donde se está ejecutando el contenedor
- `--hostname`: Escribe el valor que le pasamos como su nombre del sistema
- `--env`: Definimos las variables de ambiente, estas se definen en dependencia de la imagen que estemos utilizando
- `mysql`: Indica el nombre de la imagen con la cual instanciamos el contenedor

Al ejecutar el comando anterior, puede que veamos en consola mensajes de log sobre la creación del contenedor o podemos verlo con el comando:
`docker logs db01`

Para verificar que el contenedor está ejecutándose correctamente entonces utilizamos el comando `docker ps`. Deberíamos ver que la instancia de base de datos está corriendo.

### Conectarse a la instancia
Una vez que tenemos en ejecución la instancia podemos conectarnos a ella para verificar que todo está bien con el servicio. En contenedores lo que hacemos es ejecutar un proceso adicional al que ya está corriendo como proceso principal (main). Para lograrlo, utilizamos el comando `docker exec` especificando el proceso que queremos ejecutar y cómo. En nuestro caso, vamos a ejecutar un shell:
`docker exec -it db01 "sh"`

Básicamente le estamos diciendo *"Sabes qué, quiero conectarme al servidor `db01` con una terminal interactiva y quiero que sea el shell"*.
Con el comando anterior, el shell que especificamos se toma nuestro terminal que por lo general nos damos cuenta porque cambia el símbolo a `#`.

*Nota: El caracter `#` en bash que es el shell de Linux se utiliza para denotar que eres el administrador del sistema (mejor conocido como **root**)*

Estando conectado a la consola, podemos ejecutar cualquier comando soportado por el shell como `hostname` para visualizar el nombre del sistema o `ip add sh` para visualizar la configuración de las interfaces de red.
Lo que nos interesa es poder conectarnos a la instancia de base de datos. Lo hacemos por medio de la utilidad `mysql` de la siguiente manera:
`mysql -uroot -p`, al entrar entonces nos pedirá la contraseña, esta es la que declaramos cuando ejecutamos la instancia de contenedor.
Lo que encontraremos es el mensaje de bienvenida de la utilidad y nuestro prompt debería cambiar a `mysql>` donde podemos ejecutar sentencias SQL soportadas por el motor.

### Configurar la instancia
Conectados con la utilidad MySQL client, podemos consultar y configurar la instancia. Una de las primeras cosas que debemos conocer es cómo funciona la plataforma de base de datos. En caso de MySQL es seguro por defecto y con el usuario que tengamos solo podemos conectarnos localmente, es decir, ingresar al servidor de DB y desde allí lanzar la utilidad para conectarnos. Esto lo podemos confirmar visualizando el valor de la llave `host` con la sentencia:
`mysql> select * from mysql.user where user='root'\G;`

Lo que deberíamos observar es el valor `localhost` tal como lo anticipábamos. Podemos actualizar el valor, pero debemos tener en cuenta que el usuario `root` debería tener acceso local únicamente por razones de seguridad. Así, evitamos que personas estén probando contraseñas para el usuario de forma remota.

Algo importante es conocer dónde encontramos archivos de configuración y directorios de almacenamiento de información. En MySQL, los archivos de configuración los podemos encontrar en `/etc/mysql` o `/etc/mysql/conf.d` y el archivo de interés es `my.cnf` o `mysql.cnf`. 

#### Creación de usuario
Para crear el usuario podemos utilizar la sentencia `CREATE USER` de la siguiente manera:
`mysql> CREATE USER 'dbadmin'@'%' identified with native_mysql_password by 'test'`

Un comando equivalente al anterior es `mysql> CREATE USER 'dbadmin' identified by 'test'` por lo que la parte `@'%'` esta implicita. Ahora bien, la parte `@` la utilizamos para designar desde es posible conectarse le usuario. Luego del símbolo podemos utilizar el nombre de host, dirección IP, o hasta un segmento de red. Pueden visitar el siguiente [enlace](https://dev.mysql.com/doc/refman/5.7/en/account-names.html) para ver las opciones.

Luego de haber creado el usuario, debemos decirle qué puede hacer este usuario. A esta parte se le denomina autorización en seguridad y al hecho de haber creado el usuario se le llama autenticación. Para otorgarle permisos al usuario, en MySQL se utiliza la sentencia `GRANT` seguido del privilegio que necesitamos otorgarle. Antes de hacerlo, es necesario conocer que los permisos están asociados con roles y alcance (scope). El rol es una serie de permisos sobre acciones que un usuario puede hacer. El alcance tiene que ver con el nivel al cual se le asigna el permiso. 

Entre los permisos encontramos:
- ALTER
- CREATE
- DELETE
- DROP 

Pueden inferir de qué se trata cada uno de ellos. En cuanto al alcance se refiere podemos encontrar:
- Globales (instancia)
- Base de datos
- Tabla
- Columna 

Vamos a crear una base de datos y asignarle todos los permisos al usuario sobre esa base. Lo hacemos por medio de los comandos siguientes:

`mysql> CREATE database db_usuario1`

`mysql> GRANT ALL ON db_usuario1.* TO 'dbadmin';`

### Conectarse remotamente a la instancia
Una vez que hemos realizado las configuraciones es momento de verificar que estas funcionan correctamente. Para ello, ejecutamos un segundo contenedor que nos servirá como cliente. 
`docker run -it --name cliente_db debian "sh"`

El comando anterior creará el contenedor cliente y nos conectará a el por medio de consola. Aquí, vamos a instalar la utilidad CLI MySQL para conectarnos a la instancia de base de datos. Para ello, debemos instalarla con el siguiente comando:
`apt-get update && apt-get install default-mysql-client`

Al finalizar el comando, tendremos la utilidad instalada. Resta conectarse a la instancia con el siguiente comando:
`mysql -udbadmin -p -h172.17.0.2`

En el caso del parámetro `-h` le estamos diciendo que a ese host queremos conectarnos. Si el servidor de base de datos es el primero que ejecutaste, entonces la IP debería ser la misma que se presenta en el comando. De lo contrario, podemos investigar la dirección IP de la instacia con el siguiente comando:
`docker inspect --format '{{json .NetworkSettings.Networks.bridge.IPAddress}}' db01`

Como resultado deberíamos ver la dirección IP del contenedor de la instancia de base de datos.

Al conectarnos con mysql cli, deberíamos poder seleccionar la base de datos `db_usuario1` y crear objetos dentro de esta. Pueden hacer las pruebas respectivas para interactuar con el motor.
