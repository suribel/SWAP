EJ 1

La forma mas fácil de obtener el numero de peticiones por segundo es a través de alguna herramientas de monitorización como netstat o también con algún benchmark.
Se puede hacer a mano obteniendo diferentes estados de las conexiones. pej si tenemos nginx con el  modulo HttpStubStatusModule obtendremos alguna información como:conexiones aceptadas, conexiones abiertas....
Simplemente con  hacer la siguiente operacion tendremos el numero de conexiones por segundo :handles requests / handled connections .



EJ 2

Monitorización a nivel de sistema:

/proc

Es una carpeta en RAM utilizada por el núcleo de Unix como interfaz con las estructuras de datos del kernel.
La mayoría de los monitores de Unix usan como fuente de información este directorio.
Acceder a información global sobre el S.O. de cada uno de los procesos del sistema
Acceder y, a veces, modificar algunos parámetros del kernel del S.O.

top
Muestra cada T segundos: carga media, procesos, consumo de memoria...

vmstat
Paging(paginación), swapping, interrupciones, procesos, bloques por segundo transmitidos.


Monitorización a nivel de aplicación (Profilers) :

gprof
Da información sobre el tiempo de ejecución y número de veces que se ejecuta cada función del programa.

valgrind
Junta un conjunto de herramientas para el análisis y mejora del código.




EJ 3
