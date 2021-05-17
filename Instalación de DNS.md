# Implementar un servidor DNS
El protocolo DNS es uno de los protocolos criticos para el funcionamiento de toda organización porque la mayoría de aplicaciones y servicios cuentan con el para poder trabajar. El servicio DNS se puede implementar en diferentes sistemas operativos y versiones, pero en los basados en GNU/Linux DNS se implementa a través de [BIND](https://en.wikipedia.org/wiki/BIND) y en los basados en Microsoft tienen su propia implementación.

## Instalación del servicio DNS
Vamos a instalar el servicio DNS en un servidor con una distribución debian. Vamos a instalar el servicio, así como la documentación. Vamos a ejecutar el siguiente comando desde la consola:

`apt install -y bind9 bind9-doc dnsutils`

Bind9-doc ubica la ingformación en el sistema en un formato comprimido. En el caso de dnsutils, lo instalamos porque nos dará acceso a la utilidad `dig` para hacer consultas DNS.
Una vez que el comando termine, la instalación habrá finalizado. 
Para verificar que el servicio está activo podemos ejecutar el siguiente comando:
`systemctl status bind9`

Deberíamos obtener como resultado que esta activo y corriendo. Ahora, podemos verificar si el daemon de BIND9 está respondiendo. Utilizamos el comando dig de la siguiente manera:

`dig @127.0.0.1 . NS`

El resultado nos dice que tenemos un sistema debian, un servidor encontrado, y esta corriendo DNS. Y luego nos muestra la jerarquía completa DNS de servidores raíz y sus direcciones IP, tanto versión 4 como versión 6. Finalmente, vemos que es un servidor DNS, en 127.0.0.1 y el puerto 53. De esta forma verificamos que todo esta funcionando correctamente.

# Configuración de DNS
Una configuración que debemos cambiar se encuentra en el archivo local ubicado en `/etc/resolv.conf`. Abrir el archivo con cualquier editor de texto (nano, vi, o vim). En este caso, podemos hacerlo con vim así:

`vim /etc/resolv.conf`

Aquí puede que el servidor esté apuntado a un servidor de nombres, pero nosotros debemos asegurarnos que sea la dirección de nuestro servidor DNS en cuestión. Una buena práctica es que no pongamos la dirección local (127.0.0.1) si no la dirección de la interfaz que tenga asignada el servidor. Una vez realizado el cambio, guarde y salir. Reinicie el sistema.

## Archivos de configuración
Luego de la instalación y configuración inicial, es necesario familiarizarse con algunos de los archivos configuración. Estos archivos residen en el directorio `/etc/bind/`

### Archivo /etc/bind/named.conf
Este archivo es el archivo principal de configuración para el servicio DNS. Debemos prestar atención a las directivas que indican en qué interfaz debe escuchar el DNS, si la recursión está habilitada, y los transportadores (servidores DNS a los cuales pasar las consultas DNS no locales).

```
options {
    directory "/var/cache/bind";

    recursion yes;
    listen-on { any; };

    forwarders {
            8.8.8.8;
            8.8.4.4;
    };
};
```
<center>Archivo <b>named.conf</b> de ejemplo</center>

Así, en el archivo de ejemplo tenemos habilitada la recursión, hemos configurado que el servidor escuche en cualquier interfaz existente y que los transportadores sean los servidores DNS públicos de Google.

### Archivo /etc/bind/named.conf.options
Como su nombre lo indica, este archivo contiene todas las opciones de configuración para el servidor DNS.

```
options {
        directory "/var/cache/bind";

        // Exchange port between DNS servers
        query-source address * port *;

        // Transmit requests to 192.168.1.1 if
        // this server doesn't know how to resolve them
        forward only;
        forwarders { 192.168.1.1; };

        auth-nxdomain no;    # conform to RFC1035

        // From 9.9.5 ARM, disables interfaces scanning to prevent unwanted stop listening
        interface-interval 0;
        // Listen on local interfaces only(IPV4)
        listen-on-v6 { none; };
        listen-on { 127.0.0.1; 192.168.0.1; };

        // Do not transfer the zone information to the secondary DNS
        allow-transfer { none; };

        // Accept requests for internal network only
        allow-query { internals; };

        // Allow recursive queries to the local hosts
        allow-recursion { internals; };

        // Do not make public version of BIND
        version none;
};
```

### Archivo que contiene registros
DNS esta compuesto de muchos registros o registros de recurso, que definen la información de los dominios. Se definen archivos para zonas directas y zonas inversas. El siguiente archivo muestra una zona directa que contiene recursos varios.

```
$TTL    3600
@       IN      SOA     sid.ejemplo.com. root.ejemplo.com. (
                   2007010401           ; Serial
                         3600           ; Refresh [1h]
                          600           ; Retry   [10m]
                        86400           ; Expire  [1d]
                          600 )         ; Negative Cache TTL [1h]
;
@       IN      NS      sid.ejemplo.com.
@       IN      MX      10 sid.ejemplo.com.

sid     IN      A       192.168.0.1
etch    IN      A       192.168.0.2

pop     IN      CNAME   sid
www     IN      CNAME   sid
mail    IN      CNAME   sid
```

Explicaciones para algunos de los campos:

$TTL : (Time To Live) expresa la validez de duración (en segundos) , por defecto, de la información contenida en los registros. Una vez que este tiempo expira, es necesario verificar la información.

1. Serial : Este es el número serial que se debe incrementar por cada cambio de archivo. 

2. Refresh : define el período de actualización de datos.

3. Retry : si ocurre un error durante la última actualización de datos, se repetirá al final del tiempo de reintento.

4. Expires: el servidor se considera no disponible luego de que el tiempo expira.


'A: el registro de tipo A asocia un nombre de host a una dirección IPv4.
'PTR: Esta es la resolución inversa (lo opuesto al tipo A).

Es necesario modificar los archivos de acuerdo con los requerimientos locales que tengamos en nuestro despliegue del servicio.
