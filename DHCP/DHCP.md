# Configuración servicio DHCP
## Objetivo
Desplegar y configurar el servicio DHCP. Este objetivo incluye: Crear y configurar scopes; configurar una reservación DHCP; configurar opciones DHCP.

## Términos claves
- Asignación automática: En DHCP, un método por el cual un servidor DHCP asigna permanentemente una dirección IP a un equipo cliente desde un ámbito.
- Asignación dinámica: En DHCP, un método por el cual un servidor DHCP asigna una dirección a un equipo cliente desde un ámbito, o un conjunto de direcciones IP, por una cantidad de tiempo especificada.
- Protocolo de configuración de host dinámico (DHCP): Un servicio que automáticamente conffigura la dirección del protocolo de internet y otras configuraciones TCP/IP en las computadoras en la red por la asignación de direcciones desde un pool.
- Asignación manual: En DHCP, un método por el cual el servidor DHCP asigna permanentemente una dirección IP a un equipo específico en la red.

## Notas de la lección
El propósito de esta lección es examinar cómo los servidores o sistemas operativos de red utilizan DHCP para asignar direcciones IP a los clientes en la red.

## Entendiendo DHCP
Esto cubre lo siguiente:
1. Una descripción del formato de paquete DHCP
2. Una lista de métodos de asignación de dirección IP
3. Explicar cómo los tres componentes DHCP trabajan juntos: servidor, cliente, y protocolo.
4. Explicar los tres tipos de asignación de dirección IP soportados por DHCP:
- Asignación dinámica
- Asignación automática
- Asignación manual
5. Examinar brevemente las funciones de las opciones DHCP más importantes.

## Desplegando un servidor DHCP
- Instalar el rol en el equipo/dispositivo que brindará el servicio ﻿Installation and configuration﻿ 
- Configurar el servicio
- Iniciar el servicio
- Verificar que el servidor puede asignar rentas

## ¿Por qué es importante el servicio DHCP?
Configurar manualmente los parámetros del protocolo de internet en un cliente puede ser una tarea sencilla. Lo cierto es que a medida que la cantidad de clientes crece en una red la tarea de configuración puede volverse muy tediosa. Además se vuelve propensa a errores (humanos, por supuesto) y es complicado manejar un registro de todas las direcciones IP que han sido asignadas a los clientes. Podemos resumir las ventajas de utilizar asignación de direcciones IP de forma dinámica:
1. Control del direccionamiento
2. Mobilidad
3. Utilización efectiva del direccionamiento
4. Reconfiguración
5. Migración/actualización

## REMS

DHCP esta definido por una serie de RFCs, notablemente 2131 y 2132.

Una dirección no puede ser reservada y excluida al mismo tiempo.
EL RFC 1542 presenta cómo BOOTP debería funcionar en circunstancias en la que el cliente y el servidor estan en redes IP diferentes.
Las exclusiones deben (como recomendación) definirse al momento de la creación del ámbito de esa manera las direcciones IP no son asignadas a otros clientes.
No hay manera de cambiar una reservación una vez que ha sido creada. Para lograrlo, hay que eliminar la reservación y recrearla.
Los ámbitos se desactivan cuando se quiere que los clientes paren de usar la configuración de forma inmediata.
Los servidores DHCP utilizan bases de datos Joint Engine Technology (JET) para mantener sus registros. Estas bases de datos no se deben modificar mientras el servicio esta corriendo. Los archivos de base de datos se encuentran en systemroot\system32\dhcp.
