# Introducción

En el escenario que hemos estudiado hasta ahora, si queremos acceder al servidor web o de base de datos entonces lo hacemos de forma directa. Es decir, una vez que tenemos acceso a la red solo consultamos el recurso que queremos acceder y luego abrimos la conexión TCP según corresponda. Esto se puede observar en el diagrama básico a continuación:

![2021-06-22_11-43-57](https://user-images.githubusercontent.com/47040802/122984207-fe7bac00-d359-11eb-8033-5a2e58d378aa.png)
<center><strong>Figura 1. Diagrama de red LAN básico con servicios intranet</strong></center>

Ahora bien, en el escenario podemos observar que estamos exponiendo a nivel de red el sistema de base de datos o servidor web también. En el caso de que el cliente sea uno de nosotros, un administrador, entonces es necesaria la conexión para realizar tareas administrativas remotamente. Pero ¿cómo podemos hacerlo de modo que podamos ingresar al sistema seguramente? Bueno, conozcan a la [shell segura](https://www.ssh.com/academy/ssh) o SSH.

## ¿Qué es SSH?
SSH fue desarrollado para lidiar con el inconveniente de inicios de sesión remotos seguros. Se considera a SSH como una colección de herramientas de comunicación de red que están basadas en un protocolo abierto que está guiado por el [IETF](https://www.ietf.org/). Así, SSH permite que un usuario inicie sesión en un servidor remoto, pero que la conexión sea totalmente encriptada. SSH es otro protocolo que funciona bajo la arquitectura cliente-servidor, tal como veremos en un momento cuando expliquemos el funcionamiento.

## Criptografía de llave pública
No podemos hablar de SSH sin hablar de criptografía. Y aunque no vamos a explorar este tema a profundidad, es necesario conocer básicamente cómo funciona. Trataré de explicar este concepto con una analogía. El correo electrónico tiene dos elementos importantes, la dirección de correo (o ID de email) y la contraseña. Cualquier persona puede conocer tu ID (llave pública) o enviarte un mensaje, pero no tienen acceso a tu contraseña (llave privada). La criptografía es similar. Existe una llave pública que se encuentra tanto en el servidor como en el cliente, pero la llave privada que permite la autenticación solo se encuentra en el cliente. Lo importante es que el contenido de una comunicación encriptada con la llave pública solo puede ser descrifrada con la llave privada. Cada par de llave pública/privada es única. Los mensajes donde se da el intercambio de comunicación se muestra en la siguiente imagen:

![protocolo ssh](https://www.ssh.com/hubfs/Imported_Blog_Media/SSH_simplified_protocol_diagram-2.png)
<center><strong>Figura 2. Comunicación cliente-servidor con el protocolo SSH</strong></center>

Algo interesante de destacar es que, la llave privada nunca es enviada por la red. De esta manera, si alguien intercepta las comunicaciones solo se encontrará con datos que, en teoría, le tomaría mucho tiempo en descrifrar. Para que sea aún más interesante, SSH genera una llave de sesión la cual es cambiada regularmente. Así, en caso de que alguien pudiera descrifrar la llave privada para una transmisión, le funcionaría solo para unos cuantos minutos hasta que la llave de la sesión cambie.

### Características de las llaves
Las llaves tienen una característica esencial a la que podemos denominar longitud. Esta longitud se mide en la cantidad de bits que contiene. La cantidad de bits significa el número de posibilidades o combinaciones. En la actualidad, se recomienda que para utilizar una encriptación sólida se cuente con una longitud de 2048 bits o más para llaves tipo [RSA](https://en.wikipedia.org/wiki/RSA_cryptosystem). Algo que se debe considerar al momento de elegir la longitud de una llave es que entre más larga se requiere de más procesamiento por parte del computador para validarla.

Luego de incluir el servicio SSH en el diagrama presentado al inicio, la conexión remota queda asegurada porque no exponemos al servidor de base de datos directamente en la red si no que hacemos lo que se conoce como túnel en SSH. 
![ssh](https://user-images.githubusercontent.com/47040802/122999974-27a53800-d36c-11eb-9d2e-c300619c24db.png)
<center><strong>Figura 3. Diagrama de red LAN básico con la adición de SSH</strong></center>


Esta parte es la que estaremos trabajando en el ejercicio de este tema, cuando estemos configurando uno de los servidores con SSH.