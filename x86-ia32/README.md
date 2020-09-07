# Arquitectura x86, Procesadores IA-32

## Historia

* En 1984 Intel lanza el primer procesador de 32 bits, el 80386 con su arquitectura IA-32. Aparecen fabricantes que producirán procesadores IA'32 bajo licencia de Intel, entre ellos se destaca AMD.

* A finales de los '90, Intel y Hewlett Packard desarrollan una arquitectura de 64 bits y lanzan sin éxito los procesadores Itanium e Itanium2, principalmente por no tener compatibilidad con la arquitectura anterior.

* AMD extiende los procesadores IA-32 a 64 bits, llamando la nueva arquitectura x86-64 o AMD64 y siendo un gran éxito.

* Intel debió adoptar la arquitectura de AMD64 para IA-32 y la llamó Intel 64.

### Conclusiones

* La __Compatibilidad__ fue clave del éxito de la arquitectura IA-32 pero también ha condicionado su evolución.
* El compromise de compatibilidad adoptado en 1978 significa que las decisiones de fondo adoptadas inicialmente en el diseño de la arquitectura, inevitablemente condicionarán lo que puede o no hacerse a futuro. La administración de memoria por segmentación es un clarísimo ejemplo.

## Modos de Operación

Procesadores IA-31 tienen tres modos de operación, de los cuales surgen sus capacidades e instrucciones disponibles, y un modo especial para tareas específicas del sistema:

* Modo Real
* Modo Protegido
* Modo Mantenimiento del Sistema
* Modo extendido a 64 bits (IA-32e)
  * Submodo Compatibilidad
  * Submodo 64 bits

### Modo Real

* El procesador implementa el entorno de operación del 8086, con algunas extensiones:
  * Puede utilizar registros de 32 bits.
  * Puede reconfigurar la ubicación del vector de interrupciones (IDTR).
* Desde este modo se puede pasar por software al Modo Protegido o al Modo Mantenimiento del Sistema.
* Modo de arranque de cualquier procesador IA-32 e Intel 64 actual y futuro debido a la compatibilidad, aunque nadie trabaja en este modo.

### Modo Protegido

* Se implementa __multitasking__.
* Modo por excelencia de los procesadores de esta familia.
* Se introduce un sub-modo al que puede ponerse a una determinada tarea, denominado Virtual-8086 que permite a un programa diseñado para ejecutarse en un procesador 8086, poder ejecutarse como una tarea en Modo Protegido. Actualmente ya no es utilizado.

### Modo Mantenimiento del Sistema

* El procesador ingresa a este modo por dos caminos:
  * Activación de la señal de interrupción #SMM.
  * Mediante un mensaje SMI desde su APIC local (Interrupciones y SMP).
* Este modo fue introducido para realizar funciones específicas para la plataforma de hardware en la cual se desempaña el procesador, como lo son ahorro de energía y seguridad.
* Al ingresar a este modo, el procesador resguarda en forma automática el contexto completo de la tarea o programa interrumpido, y pasa a ejecutar en un espacio seaprado. Una vez efectuadas las operaciones necesarias y cuando debe salir de este modo, el procesador reasume la tarea o programa interrumpida en el modo de operación en el que se encontraba.

### Modo extendido a 64bits (IA-32e)

* Los procesadores Intel 64 además de los modos de trabajo de IA-32, incluyen un modo IA-32e en el que se active la arquitectura de 64 bits.
* Para pasar a este modo, el procesador debe:
  * Estar trabajando en Modo Protegido
  * Tener Paginación Habilitada
  * PAE (Page Address Extention) activo
* IA-32e a su vez cuanta con dos sub-modos.
  * Submodo Compatibilidad
  * Submodo 64 bits

#### Submodo Comptabilidad

* Pensado para garantizar la transición de 32 a 64 bits, permitiendo aplicaciones de 16 y 32 bits legacy ejecutarse sin recompilación bajo un sistema operativo de 64 bits.
  * Entorno de ejecución de la tarea de 32 bits es la de la arquitectura IA-32.
  * Kernel no soporta el manejo de tareas del modo IA-32.
  * Sub-modo Virtual 8086 desaparece.
  * Incluye mecanismos de protección del modo 64 bits.
* Sistema Operativo de 64 bits puede ejecutar tareas de 16 y 32 bits junto con otras de 64 bits, sobre la base de diferentes segmentos de código. La tarea de 32 bits accede a una arquitectura IA-32 pura, utilizando direcciones de 16 y 32 bits, con 4Gbytes de espacio de direccionamiento y con la posibilidad de acceder por encima de ese límite habilitando PAE.

#### Submodo 64 bits

* Este modo habilita un Sistema Operativo de 64 bits a ejecutar tareas escritas utilizado direcciones lineales de 64 bits.
* Se extienden de 8 a 16 los Registros de propósito general, cuyo ancho de palabra ahora es de 64 bits, agregándose los registros R8 a R15.
* Se introduce el prefijo `REX` para las instrucciones que deseen acceder a las versiones de estos registros de 64 bits.
* Los registros SIMD también se extienden a 16 manteniendo su ancho de 18 bits, XMM0 a XMM15.

## Modelo del Programador de Aplicaciones

### Arquitectura de 16 bits básica

#### Registros y Espacio de Direccionamiento

![registros](./registros16bit.png)

* AX: Acumulador
* BX: Base
* CX: Contador
* DX: Data
* SourceIndex y DestinationIndex: Enteros, generalmente utilizados como punteros a memoria.
* BasePointer y StackPointer: Punteros de pila.
* CodeSegment, StackS, DataS, ExtraS: Registros de segmentos.

#### Floating Point Unit

FPU es una unidad donde podemos desarrollar aritmética más avanzada, con números reales en notación científica.

Tiene 8 registros R0-R7 de 80 bits de ancho y registros de control.

Como no entraba en el 8086, se entregaba en conjunto con un chip separado. Más adelante, con el 486 de 32 bits, la tecnología avanzó como para que los microprocesadores integren su propia FPU.

Utilizan como datos el standard [IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) de Extended Double, Double Precision & Single Precision.

### Arquitectura 32 bits básica

#### Registros y Espacio de Direccionamiento

![registros](./registros32bit.png)

Convierte los registros a contener palabras de 32 bits, pero mantiene compatibilidad siendo que los 16 bits menos significativos de los registros, siguen pudiendo ser referenciados por segmentos de 8 bits (AX, BX, CX, DX).

#### Extensiones MMX (Multimedia Extensions)

![mmx](./mmx.png)

Con el tiempo, se comenzó a integrar, a la arquitectura y organización, recursos para procesar algoritmos de video y audio (Digital Signal Processing).

Desde el punto de vista de la arquitectura, se agregarion los registros MMX (Multimedia Extensions) de 64 bits cada uno, MM0-MM7 que permiten __data level parallelism__. Esto "empaquetar" words de distinto tamaño y procesarlas como una simple instrucción, por ejemplo, procesando en vez de un número de 64 bits, 8 números de 8 bits en una misma instrucción.

Estos registros funcionan como ALIAS de los 64 bits menos significativos de los 8 registros de la FPU. Entonces la limitación, es que __no se pueden mezclar instrucciones de la FPU con MMX__, porque cada vez que quieras pasar de MMX a FPU, hay que correr una instrucción que restaure los registros.

#### Extensiones XMM

![xmm](./xmm.png)

Registros __independientes__ de 128 bits para procesamiento vectorial (__Data Level Parallelism__)

Se introdujeron con la llegada del __Pentium III__.

