# MEMCACHED

Autores: Mario Antonio López Ruiz, Antonio Manuel Rodríguez Martos

## ¿Qué es y como se utiliza Memcached?

Sistema distribuido de propósito general para caché que se instala en un servidor y funciona como un servicio más de la máquina. Este servidor es completamente independiente de PHP y se puede usar en cualquier lenguaje de programación que tenga librerías para interactuar con él. Reduce las necesidades de acceso a un origen de datos externo. Actualmente es muy utilizada por múltiples sitios web, algunos de los más activos y visitados de la red, como Youtube, Reddit, Facebook y Twitter.

El concepto es sencillo, si algo cambia poco, se guarda ya procesado y listo para servir sin necesidad de acceder a la base de datos y procesarlos. Es bastante habitual, además de guardar datos propios de las consultas del usuario, almacenar datos de sesión en memcached en vez de en disco (por defecto PHP los guarda en disco). Con ello, podemos tener múltiples servidores balanceados sirviendo el mismo código, y todos utilizando memcached para las sesiones sin tener que configurar una carpeta de disco compartida por todos.

## ¿Cómo funciona?

Almacena en la memoria RAM una copia cacheada de los elementos, acelerando el acceso a ellos y reduciendo las consultas al disco duro, lo cual es muy lento. Los datos se almacenan en una tabla hash, mapeando según la clave que se le asocie a dicho ítem. Permite mantener un pool de Memcached Servers, para momentos en donde la cantidad de conexiones no pueda ser gestionada por un único servidor, y poder así balancear la carga de conexiones.

Para poder utilizarlo hay que hacer uso de las librerías de memcached, que poseen los siguientes métodos básicos:

 - Connect: Información relativa a la realización de la conexión
 - SetData: Envío de datos a guardar (pueden ser arrays u objetos, la librería se encarga de serializar), tiempo de cacheado y una clave alfanumérica-
 - GetData: Pasando la clave a recuperar, se devuelven los datos unserializados.
 - FlushData: Eliminta datos relativos a la clave enviada.
 - FlushAll: Elimina todas las claves de memcached.

La estrategia de funcionamiento consta de los siguientes pasos:

 - Antes de hacer una consulta a BBDD, generamos una clave y miramos si el dato está ya cacheado.
 - En caso afirmativo, la consulta no se realiza, y se devuelven los datos cacheados.
 - En caso negativo, se realiza la consulta, los datos se cachen y se devuelve.

## Arquitectura

Utiliza una arquitectura cliente-servidor. Los servidores mantienen una tabla hash (array asociativo clave-calor), añadiendo los clientes datos al array y accediendo a él. Dichas claves pueden tener una longuitud máxima de 250 bytes y los datos un tamaño máximo de 1 megabyte. 

Los clientes utilizan por defecto para conectarse el puerto 11211, manteniendo una lista de todos los servidores. Los servidores no se comunica entre ellos. Si un cliente desea establecer o leer el valor correspondiente de cierta clave, se realiza un cálculo mediante un algoritmo hash para determinar el servidor que se va a utilizar, entonces se pone en contacto con el servidor y éste usará otro hash para determinar dónde almacenar o leer el valor correspondiente.

Los clientes deben tratar Memcached como una caché transitoria, no pueden asumir que los datos almacenados en Memcached estarán ahó cuando los necesites (esto es debido a que los servidores mantienen los valores en RAM, una vez a gotada su memoria, se van descartando los valores más antiguos). Hay distintas solucionar para proporcionar un almacenamiento permanente de dichos valores, como MemcacheDB o Membase de Northscale.

![img](http://4.bp.blogspot.com/-_lol17ifetM/VW8DqGWXoAI/AAAAAAAAABk/eZMnifWyZYo/s320/Arquitectura.png)

## Desventajas

Por cada clave se permite almacenar 1Mb de datos serializados, es un tamaño considerablemente grande, pero si insertamos un valor mayor no se devuelve una excepción, sino que devuelve simplemente "false".

El servidor memcached tiene definida una memoria de uso, si cacheamos muchas casos se puede producir un overflow y las claves más antiguas comienzan a expirar.

Hasta que está todo cacheado puede darse el caso de muchas peticiones intentando cachear lo mismo de forma simultánea. Este fenómeno se conoce como caché storming, y puede provocar problemas en aplicaciones al realizar muchas peticiones de forma simultánea.

Memcached no lleva autenticación, cualquiera con acceso al servidor si conoce las claves puede recuperar nuestros datos, por ello raramente se instala en servidores compartidos.

## Ventajas

Si nos decantamos por utilizar memcached observaremos que el tiempo de respuesta de nuestra aplicación web aumenta considerablemente, así como la capacidad de servicio.  La instalación es sencilla, y la librería a usar como se ha visto anteriormente no requiere del aprendizaje de muchos métodos, es también simple. 

## Seguridad

Un fallo importante de Memcached está relacionado con el soporte proporcionado al protocolo UDP. Cloudfare comenzó a registrar un comportamiento extraño en los servidores Memcached, que por UDP recibían paquete de pocos bytes, algo poco significativo, pero el servicio proporciona una respuesta de tamaño mucho mayor. El problema entonces no reside en atacar Memcached, sino de utilizar memcached como herramienta para atacar, direccionando IPs concretas como respuesta.

Exactamente, los paquetes registrados poseían un tamaño de 15 Bytes, y la respuesta de los servidores era de paquetes de 750 Kb aproximadamente, con una capacidad de amplificación de hasta los 257 Gbps

Al ser esta una de las soluciones más populares de cacheo entre los servicios de Internet, la magnitud del problema aumenta considerablemente.

###### El ataque DDoS mas grande jamás registrado

GutHub recibió un ataque DDoS que generó un tráfico de 1,3 Tbps, convirtiéndose en el mayor ataque DDoS registrado de la historia. Memcached está pensado para sistemas que no están conectados a Internet, pero esto no ocurre en la realidad. Akamai realiza una estimación de que hay al menos 50.000 conectados a la red, siendo vulnerables. El factor de amplificación puede ser de más de 50.000 veces, por ejemplo, una solicitud de 203 bytes puede convertirse en una respuesta de 100 MB.

![img](https://media.wired.com/photos/5a980877d2c5f03d18982b24/master/w_885,c_limit/Akamai_graph.jpg)

El ataque fue mitigado rápidamente, filtrando todo el tráfico del puerto UDP 11211. Esto se planteó así ya que el fallo de seguridad era conocido antes de que nadie lo llevase a cabo, además de una rápida actuación.

Otra posible solución además de filtrar las peticiones por el puerto UDP, sería que no existan reflectores expuestos a Internet, aunque es algo realmente complejo.

Probablemente no sea el último ataque de este tipo utilizando Memcache como herramienta, ya que últimamente se han detectado escaneos de puertos en los servidores con memcached abiertos.