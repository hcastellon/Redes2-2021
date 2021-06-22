# Sistema de Nombres de Dominio
El Sistema de Nombres de dominio (o DNS) es uno de los servicios de infraestructura y protocolo más críticos que existen en telecomunicaciones. Si este servicio no está disponible, internet no funciona prácticamente o al menos los servicios que dependen de el. Poder desplegar y administrar este servicio es una habilidad esencial para un administrador de sistemas o roles similares.

![It's always DNS](https://www.cyberciti.biz/media/new/cms/2017/04/Its-not-DNS.-There-is-no-wayits-DNS.-It-was-DNS.jpeg)
<center>Un famoso poema japonés utilizado cada vez que hay un problema DNS</center>

## ¿Qué es DNS?
En palabras simples, DNS proporciona una forma de traducir y/o mapear direcciones IP a nombres de dominio o viceversa. El objetivo de hacer esa traducción es que nosotros como usuarios podamos administrar de una forma más fácil y útil porque trabajar con nombres es más amigable. Ahora bien, DNS también tiene otras ventajas y no solo que sea más conveniente para nosotros. Una de ellas es que las direcciones IP pueden cambiar o reasignarse con el tiempo, porque se esta haciendo una migración, porque cambiaste de proveedor de servicios de internet, u otra razón, entonces con DNS permite actualizar el mapeo de modo que el nombre no cambia. Esto se traduce en transparencia al usuario que utiliza un determinado servicio.

## Un poco de historia (no tan aburrida 😉)
La primera forma de DNS surge en la década de los 70, cuando el mapeo de direcciones IP a nombres se hacia con un archivo en texto plano llamado **hosts.txt** que era distribuido por ftp a otras máquinas en la internet. El [**<u>Network Information Center</u>**](https://www.internic.net/) (*o Centro de Información de Red*) era el encargado de mantener este archivo y distribuirlo a todo ARPANET.

A medida que la cantidad de máquinas fue creciendo, se dieron cuenta que mantener un archivo así era algo insostenible. Entonces pensaron en crear un sistema distribuido en el cada sitio mantuviera información acerca de los equipos. Con sitio nos referimos a redes que en ese momento eran campus de universidades. Una de esos equipos en cada sitio se tenía que considerar autoritativo, y la dirección de ese equipo se encontraría en una tabla maestra que sería consultada por los otros sitios. A este dispositivo autoritativo es lo que conocemos como servidor DNS y a estos, especialmente, [servidores DNS raíz](https://www.iana.org/domains/root/servers).

Para conocer más acerca de DNS, puedes consultar [esta lista de RFCs](https://powerdns.org/dns-camel/) relacionados al protocolo. 

## Elementos de DNS
Vamos a explorar brevemente los elementos que componen a DNS, de manera podamos entender cómo funciona y qué considerar al momento de implementar el rol.

### El archivo Hosts
Como vimos anteriormente, un archivo en texto plano era el utilizado antes que DNS emergiera como sistema y protocolo. Algo interesante es que ese archivo aún se mantiene y utiliza, con la excepción que ahora es local. Es decir, casi cualquier sistema operativo viene con este archivo que se utiliza como primer mecanismo de resolución al momento de hacer consultas DNS. Por ejemplo, en sistemas operativos Windows el archivo Hosts se encuentra en la ruta `C:\Windows\System32\drivers\etc`. En los sistemas Linux/Unix, esta tabla (como también se le conoce) se localiza en `/etc/hosts`.

Estos archivos son simples de entender porque en la primera columna se encuentra la dirección IP y en la segunda columna encontramos los nombres de hosts.

### Convenciones de nomenclatura de host y dominio
Nosotros utilizamos DNS de forma rutinaria. Si abrimos un navegador y escribimos `www.google.com` estamos haciendo uso de DNS. Lo que escribimos en el navegador se conoce como **Fully Qualified Domain Name** (FQDN de forma abreviada). Es necesario prestar atención a la estructura porque cada punto tiene un significado fundamental. Los FQDN se leen de derecha a izquierda porque DNS se considera un árbol invertido, en el cual la raíz está en el nivel más alto y las ramificaciones estan hacia abajo. Así, **com** es el dominio de alto nivel, **google** es el dominio de segundo nivel y **www** solo indica el protocolo de comunicación.

#### Dominios de alto nivel (TLDs)
Los TLDs son como las primeras hojas del árbol. Estos proporcionan la organización categórica del espacio de nombres DNS. En palabras simples, el espacio ha sido dividido en categorías que responden a usos específicos. Ejemplos de estos usos puede ser por geografía o función. En la actualidad existen un sinnúmero de TLDs y pueden ser agrupados de la siguiente manera:

- TLDs con código de país: .ni, .us, .uk, .cr, .ca, .co
- TLDs genéricos: .com, .org, .net, .edu, .mil
- TLDs de marcas: .linux, .microsoft, .unicef

Los TLDs, por lo general, se registran con lo que se conoce como *Domain registrar*. 

#### Dominios de segundo nivel (SLDs)
Los SLDs se crean dentro los límites del espacio de nombres organizacional. Estos funcionan una vez que hemos registrado un dominio. Las compañías, comunidades educativas, ONGs, entre otros adquieren nombre únicos en este nivel. **redhat.com** y  **ubuntu.com** son ejemplos de SLDs, donde redhat y ubuntu los representan específicamente.

#### Dominios de tercer nivel (tLDs)
Cuando has registrado un dominio, entonces puedes hacer lo que sea con los dominios de tercer nivel. Sin embargo, se recomienda que los tLDs reflejen los nombres de los dispositivos o cualquier otro uso funcional. 

### Tipos de servidor
#### Servidor autoritativo
El servidor autoritativo es aquel en el cual los archivos de configuración residen. Cualquier actualización a la base de datos DNS se realiza en este servidor. También conoce todo sobre los hosts y subdominios bajo el dominio principal.

#### Servidor caché
Como su nombre lo indica, solo realiza caché. No contienen archivos de configuración para ningún dominio en particular. En lugar de eso, cuando un cliente consulta a un servidor caché este verifica si existe una entrada para la consulta. Si no hay una coincidencia, encontrará al servidor indicado al cual redireccionar la consulta. La respuesta se guarda en el caché para la próxima vez que se repita la misma consulta.

*Nota: Leer acerca de tipos de registros que se utilizan en DNS. Pueden utilizar [este recurso](https://support.google.com/a/answer/48090?hl=es)*

