# Laboratorio #2 - Administración de Redes 
## Integrantes
Juan José Cifuentes Cuellar

Tomas Candelo Montoya

Carlos Farouk Abdalá Rincón 

****
# Planificación
Para el desarrollo del proyecto, en un primer lugar tuvimos que realizar una labor de planeación en la que definiríamos todos los parámetros generales de la red a fin de facilitar el desarrollo más adelante. Para ello, y en primer lugar, decidimos guiarnos por las preguntas planteadas por los desafíos asignados a lo largo del corte para definir las generalidades del direccionamiento y enrutamiento requeridos. Las preguntas se encuentran a continuación.

## Preguntas

- **¿Cuántas subredes se necesitan?**  

Cada uno de los espacios de direccionamiento que nos dan tiene un número diferente de subredes necesarias, pero las generalidades son las siguientes:

• Las dos redes correspondientes a las Intranets necesitan un total de **3** subredes: 2 para las VLANs productivas y 1 para la VLAN troncal.

• La red definida para la conexión DMZ e ISP en Bogotá necesita **2** subredes: 1 correspondiente al link que tiene el router de la intranet con su respectivo ISP y otra para la red demilitarizada.

• La red definida para la conexión ISP en Madrid solo necesita **1** subred: La subred necesaria para el link entre el router de la intranet y el ISP en Madrid.

• La red disponible para las direcciones loopback tan solo necesita **1** subred capaz de dar una dirección a todos los routers de la topología.

• La red IPv4 definida para el Internet necesita de un total de **5** subredes, cada una asignada a los diferentes links que se crean entre los routers.

- **¿Cuántos host/interfaces necesita cada subred?** 

Gracias a que la gran mayoría de estos espacios son IPv6, no tenemos que preocuparnos por ello, puesto que todas las redes tendrán el mismo tamaño de 2^64 hosts, permitiendonos ignorar por completo este requisito.

Sin embargo, para las redes IPv4 que tenemos que calcular, si es importante mencionar que cada una necesita un mínimo de **2** hosts, uno por cada interfaz de la conexión montada.

- **¿Qué dispositivos se encuentran en cada subred?**

La disposición de los dispositivos en las diferentes subredes es la siguiente:

• Los PC1-4, los Switches_Intranet1-3 y la interfaz del R1_BOG que se conecta a ellos pertenecen a las diferentes VLANs de la Intranet Bogotá, con los dispositivos intermedios en la VLAN 99 y los hosts en las diferentes VLANs funcionales según lo indicado en la guía (50 y 100).

• Los PC5-8, los Switches_Intranet4-5, el Multilayer-Switch0 y la interfaz del R2_ESP que se conecta a ellos pertenecen a las diferentes VLANs de la Intranet Madrid, con los dispositivos intermedios en la VLAN 1 y los hosts en las diferentes VLANs funcionales según lo indicado en la guía (25 y 100).

• Los servidores Web y DNS, el Switch_DMZ y la interfaz del R1_BOG que se conecta a ellos pertenecen a la subred DMZ.

• Las conexiones generadas entre los R1_BOG y R2_ESP y sus respectivos ISP pertenecen cada una a las subredes alocadas a esta función.

• Los routers que componen el Internet, es decir, ISP_FL, ISP_NET y las interfaces correspondientes de ISP_ESP e ISP_BOG pertenecen a las diferentes subredes IPv4 preparadas para cada una de sus diferentes conexiones.

- **¿Qué partes de la red utilizan direcciones publicas y que partes direcciones privadas?**  

Una vez más, esta pregunta deja de tener sentido cuando hablamos de direccionamiento IPv6, pues esta distinción ya no es necesario a fin de ahorrar en número de direcciones disponibles. Si entendemos esta distinción sin embargo como que partes de la red necesitan una dirección GUA o LLA, entonces nos encontramos con que todos los dispositivos e interfaces que funcionen con el protocolo IPv6 utilizan tanto "públicas" como "privadas".

En cuanto al direccionamiento que ocurre con IPv4, todas las conexiones son de tipo público, pues se refieren a aquellas redes que componen la parte pública del Internet, sin llegar a entrar a una LAN en ningún momento.

- **¿Dónde deberían estar conservadas estas direcciones?** 

Los servidores Web y DNS, Switches de las intranets (incluyendo el MLSW) y Routers deberían mantener siempre sus direcciones de forma estática, para no generar problemas con las puertas de enlace (routers), para permitir que todos los hosts puedan acceder a los diferentes servicios de la red de forma confiable (servers) y para mantener un sistema organizado respecto al ruteo de las VLANs truncales 99 y 1 (switches).

Los PC pueden cambiar de dirección en cualquier momento, pues esto no afecta el funcionamiento de la red.

- **¿Se requiere una asignación dinámica y/o estática? ¿Dónde?**

En base a lo anterior, podemos ver que los dispositivos intermedios requieren de una asignación de IP estática, mientras que los dispositivos hosts, como los PC, requieren de una asignación dinámica. Este direccionamiento dinámico será de tipo stateless DHCPv6 por razones que trataremos más adelante.

- **¿En que terminal se configuraron los servicios requeridos?**

Los servicios de direccionamiento dinámico se configuraron en las interfaces de los routers R1_BOG y R2_ESP conectadas a sus respectivas intranets, puesto que son solo los PC quienes necesitan de este servicio.

- **¿Qué servicio de IPv6 se debe configurar para para limitar el acceso al servidor web por el puerto 80 y 443 en las vlans especificas?**

Se debe utilizar algún servicio de filtrado de paquetes. Por razones que discutiremos más adelante, el servicio escogido fue el de Access List, aunque vale la pena mencionar que se configuró en su versión IPv6, no IPv4. Esto debido a que la parte de la red con direccionamiento IPv4 no tiene ningún host al que denegar el acceso al servidor Web.

- **¿Dónde se deben ubicar los ACLs?**

La forma en la que planteamos nuestra solución requiere que las Access Lists se apliquen en las interfaces de salida de los routers conectados a cada intranet, es decir, en la interfaz FastEthernet0/0 del R1_BOG y en la interfaz Serial0/0/0 del R2_ESP

- **¿En qué interfaces se deben configurar OSPF o EIGRP (no RIP) y/o rutas estáticas y redistribución entre protocolos?**

Gracias a que la topología nos especifica las diferentes partes de la red en las que se deben aplicar los diferentes protocolos de enrutamiento, es fácil identificar las interfaces que requiere cada protocolo. Solo hay una conexión que queda a elección nuestra, y esta es la conección entre ISP_FL e ISP_NET, la cual terminamos eligiendo seria EIGRP.

• La interfaz serial del R1_BOG y la interfaz a la que se conecta en el ISP_BOG llevan el protocolo OSPFv3.

• La interfaz serial del R2_ESP y la interfaz a la que se conecta en el ISP_ESP llevan el protocolo EIGRPv6.

• Las interfaces del ISP_FL en su totalidad y las interfaces a las que se conecta en el ISP_BOG, ISP_ESP e ISP_NET llevan el protocolo EIGRP.

• Las interfaces del ISP_BOG e ISP_ESP que conectan al ISP_NET junto a las respectvas interfaces a las que se conectan llevan el protocolo OSPF.

Ahora bien, para la redistribución entre protocolos nos limitamos únicamente a aquellos routers que comparten más de un 

- **¿Qué servicio(s) de migración se debe(n) implementar para permitir el acceso al servidor Web instalado en el DMZ configurado completamente en IPv6?**

Para el desarrollo de este proyecto decidimos utilizar el servicio de migración Tunneling, por lo que creamos un tunel virtual entre los routers ISP. Vale la pena mencionar que para el funcionamiento de este decidimos aprovecharnos del tamaño reducido del laboratorio y utilizar enrutamiento estático a fin de que funcionara esta conexión.

## Subneteo

La red empresarial cuenta con un total de 5 espacios de red, cuyas generalidades ya mencionamos anteriormente. En concreto, cada uno de los espacios de direccionamiento consiste de_

• La intranet de Bogotá 2001:1200:A1::/48 para un total de 3 vlans (50 Ing, 100 Tech, 99 native)

• La intranet de Madrid 2001:1200:B1::/48 para un total de 3 vlans (25 Tesoreria, 100 Vice, 1 native)

• La Conexión del Router de INTRANET Bogotá con el ISP de la misma ciudad y el espacio DMZ de la red (que cuenta con un total de 6 ) 2001:1200:C1::/48 

• La Conexión del Router de INTRANET España con el ISP del mismo pais 2001:1200:D1::/48 

• Las Conexiones entre los routers que hacen parte del espacio de internet, este espacio de direcciones a diferencia de los cuatro anteriores es un espacio IPv4, con el siguiente rango de red 190.85.201.0/24

Para el proceso de subneteo en las redes IPv6 no se tiene informacion de cuantos host se necesitan por cada subred por lo que su proceso de subneteo se basa en que estamos buscando direcciones con un inteface ID de longitud 64, mientras que para el subneteo del espacio IPv4 se tiene en cuenta que necesitamos 2 host por cada subred (conexion entre routers) y se tiene un total de 5 conexiónes. 

**Subneteo de la intranet de INTRANET Bogotá**

![SUBNETEO DE LA INTRANET DE INTRANETOTÁ](/image/Subneto_inteNet_Bog.png)

**Subneteo de la intranet de Madrid**

![SUBNETEO DE LA INTRANET DE MADRID](/image/Subneto_inteNet_MAD.png)

**Subneteo del DMZ y WAN entre R1_INTRANET y ISP_INTRANET**

![SUBNETO DE LA RED DMZ Y WAN ENTRE R1_INTRANET Y ISP_INTRANET](/image/Subneteo_DMZ_R1-ISP.png)

**Subneteo del ISP_ESP**

![SUBNETO DE LA RED ISP DE ESPAÑA](/image/Subneteo_R2-ISP_MAD.png)


Para el subneteo de los espacios de red IPv6 se realizo el mismo proceso para los 4 espacios, como estandar buscamos una longitud de interfaz de 64 bits por lo que en este caso el número de bits que hacen falta para cumplir este estandar es de 16 bits (64-48=16) lo que seria igual a un cuarteto de la dirección el cual va a ser nuestro identificador de subred. Una vez con esto sacamos las primeras 15 posibles subredes para cada espacio, unicamente para ver las primeras posiblidades que tenemos y solo seleccionamos la cantidad de subredes necesarias por el espacio de red:

• La intranet de Bogota necesita un total de 3 subredes

• La intranet de Madrid de igual manera necesita 3 subredes

• El DMZ y la conexión WAN entre el R1_BOG y el ISP_INTRANET necesita 2 subredes 

•La conexión Wan entre el R2_ESP y el ISP_ESP necesita una sola subred

**Subneteo del espacio WAN**

![SUBNETO DEL ESPACIO WAN(INTERNET)](/image/Subneto_internet.png)

Para este Subneteo el número de host pedidos por las especificaciones de la red es de 2 lo que da un total de 2 bits de reserva, lo que da una nueva mascara de subred con identificador 30 y un incremento de 4 en el cuarto octeto de la dirección. En este caso son necesarios 5 rangos de red para las conexiones entre los routers que pertenecen al internet

## Tabla de direccionamiento

Finalmente con los Subneteos terminados pasamos a la construcción de la tabla de enrutamiento en la cual asignamos las direcciones IP a los diferentes dispositivos de la red y en los puertos que deben de ser asignadas, el resultado de esas asignaciones fue la siguiente tabla:

![Tabla de direccionamiento en la red empresarial](/image/TABLA_DIRECCIONAMIENTO.png)

****
# Discusión de alternativas

## Direccionamiento
Como sabemos, no hay alternativas viables para la asiganción de direccionamiento estático, por lo que para gran parte de nuestra topología es imperativo que nos demos a la labor de asignar a mano cada una de las direcciones de nuestra tabla de enrutamiento. 

Sin embargo, cuando hablamos del direccionamiento dinámico que tienen que tener los PC, nos encontramos con 3 posibles opciones al partir del hecho de que tienen que ser dinámicas y de protocolo IPv6:

>**Opción 1:** Utilizar un servicio DHCPv6 stateful. Podríamos haber configurado en un cada uno de los routers o en un servidor DHCP la opción de que toda la información de red fuera entregada al host, de forma que se llevara la cuenta de las direcciones que se van asignando. Sin embargo, en la topología de la guía de laboratorio no se nos presenta ningún servidor que podamos utilizar para esta labor, y decidimos que la configuración podría ser muy desgastante cuando otras opciones nos permitían tomar ventaja del hecho de que, usando SLAAC, los hosts pueden generarsu propia dirección IP. Por estos motivos, descartamos esta opción.
>
>**Opción 2:** Utilizar un servicio únicamente de SLAAC. Si bien este servicio resuelve el problema de tener que asignar todos los diferentes rangos de direccionamiento, tiene la desventaja de que no nos permite darle al host dirección de red necesaria como el servidor DNS. Viendo que para los requisitos del laboratorio esta opción es inviable, puesto que no permitiría que los hosts accedieran a la página web usando el servicio DNS, la descartamos.
>
>**Opción 3:** Utilizar un servicio DHCPv6 stateless. A pesar de que la configuración no es tan simple como la opción 2, esta opción tiene la ventaja de que permite que los diferentes hosts se hagan cargo de su propia dirección IP encima de entregarles el resto de información relevante, en este caso, dirección de servidor DNS.

Finalmente, nos decantamos por la tercera opción, por lo que configuramos la **flag O** y configuramos en cada una de las subinterfaces de las diferentes VLAN una pool de direcciones DHCP que, si bien contenía la dirección del servidor DNS, no nos exigía configurar todos los diferentes rangos de direccionamiento, permitiendo su reutilización en todas las subinterfaces independiente del rango de direccionamiento correspondiente a sus VLAN.

## Enrutamiento
Como bien mencionamos anteriormente, gran parte de los protocolos de enrutamiento son específicados por la guía de laboratorio, por lo que solo tuvimos que considerar alternativas en dos casos:

- **Conexión entre ISP_FL e ISP_NET:** Puesto que esta conexión ocurre entre los espacios definidos por la guía, estaba a nuestra decisión cual protocolo sería el encargado de aprender la información correspondiente a este link. 

    Puesto que necesitamos que el enrutamiento sea capaz de detectar cambios en la topología, decidimos que enrutar esta dirección de forma estática no era inteligente. Así, entre los 3 protocolos de direccionamiento vistos hasta el momento (RIP, EIGRP y OSPF) nos decidimos finalmente por EIGRP, puesto que al ser el protocolo que habíamos manejado anteriormente, era el protocolo con el que nos sentíamos más cómodos configurando, además claro de ser el protocolo con menor distancia administrativa entre los demás, lo cual representa una, si bien pequeña dado el tamaño de nuestra topología, ventaja al momento de escoger rutas más eficientes.

- **Conexión virtual por Tunneling:** En este caso, al considerar que el túnel mantiene siempre la misma dirección y que los posibles cambios en la topología solo podría ocurrir en el Internet, sitio que ya está enrutado de forma dinámica, nos decantamos por enrutar esta conexión de forma estática, indicando a los routers R1_BOG y R2_ESP que enrutaran paquetes desconocidos hacia su ISP y a los routers ISP_BOG e ISP_ESP que enrutaran los paquetes destinados a cada una de las respectivas intranets mediante su interfaz virtual de túnel.

## Filtrado
Para el filtrado de paquetes, y basados en lo que hemos visto durante este semestre, teníamos la opción de usar un Firewall o Access List. Sin embargo, considerando que durante el semestre nunca nos tomamos el suficiente tiempo para explorar un Firewall, sus usos concretos o su configuración, nos decantamos por las Access Lists, las cuales ya hemos configurado anteriormente, entedemos de forma plena, y nos permiten el bloqueo de paquetes no solo en base a rangos de direcciones IP, sino también en base a los puertos siendo usados, lo que nos permite cumplir más fácilmente uno de los requisitos del laboratorio.

## Migración

Considerando los métodos de migración que vimos en este corte, teníamos 3 opciones para configurar la migración de paquetes entre IPv6 e IPv4. Estas opciones eran:

>**Dual Stack:** La opción de configurar tanto IPv6 como IPv4 en cada uno de los hosts intermedios fue descartada inmediatamente por varias razones, entre las que se cuentan el desgaste de configurarlo respecto a otras opciones, falta de un rango claro de direccionamiento IPv6 que deberíamos usar para los dispositivos del Internet y la forma en la que invalida la práctica de hacer coexistir dos redes con protocolos de direccionamiento diferentes.
>
>**Tunneling:** Gracias a que la configuración de este método requiere tan solo de configurar dos routers de frontera para la creación del tunel virtual, nos decidimos por ella. Además de esto, consideramos el hecho de que, al encapsular el paquete como un paquete IPv4, puede hacer uso del enrutamiento dinámico IPv4 que ya habíamos configurado sin mayor cambio.
>
>**Translators:** Como vimos durante el semestre, este método es más bien un último recurso en caso de que no podamos o queramos aplicar cualquiera de los 2 métodos anteriores, pues no soluciona el hecho de hacer que los dos protocolos de direccionamiento coexistan en la misma topología. Por ello, y por la necesidad de escoger otro nuevo rango de direccionamiento IPv4 al cual traducir los paquetes, descartamos esta opción.

Así, terminamos configurando Tunneling como nuestro método de migración entre protocolos de direccionamiento.

****
# Configuración

Para todos los dispositivos intermedios(Routers,switches) se hace la misma configuración básica, en la que se le asigna un nombre, un mensaje de bienvenida y contraseñas de acceso tanto para entrar a su configuración (por cable de consola o por telnet) como para acceder a sus diferentes niveles administrativos.

Los comandos usados para estas configuraciones basicas fueron los siguietes

```
Switch(config)#hostname SW1_INTRANET
SW1_INTRANET(config)#line console 0
SW1_INTRANET(config-line)#password cisco
SW1_INTRANET(config-line)#login
SW1_INTRANET(config-line)#exit
SW1_INTRANET(config)#
SW1_INTRANET(config-line)#password cisco
SW1_INTRANET(config-line)#login
SW1_INTRANET(config-line)#exit
SW1_INTRANET(config)#banner motd #Se encuentra configurando el switch 1 en Bogootá. Ingrese su contrasena#
SW1_INTRANET(config)#enable password cisco
```

Ademas de esto, tanto la intranet de Bogotá como la de Madrid, tenemos que permitir el paso y uso de las direcciones IPv6 para eso a los switches se les realizó la siguiente configuración 

```
SW1_INTRANET(config) SDM PREFER DUAL-IPV4-AND-IPV6 DEFAULT
SW1_INTRANET(config) END
SW1_INTRANET# reload
```

Con esta configuración se permite que los switches puedan utilizar y reconocer direcciones IPv6 

En el caso de los routers unicamente tenemos que hacer es habilitar el propio router como un router IPv6, el cual podra enviar y recibir paquetes IPV6, rutas estaticas o protocolos de enrutamiento IPV6, el comando es el siguiente:

```
R1_BOG(config) IPv6 unicast-routing
```

## Creacion de las vlans

Lo primero que se va a hacer para la configuración de las vlans es asiganr la default gateway, la cual es la direccion vinculada con la subinterfaz del router que tiene la vlan que usaremos de troncamiento (En el caso de Bogotá la vlan 99 y en el caso de Madrid la vlan 1) como se puede ver en la tabla de enrutamiento, El comando es el siguiente:

```
SW1_INTRANET(config) ipv6 route ::/0 2001:1200:A1:1::1
```

Ahora, apagamos todas sus interfaces.
```
SW1_INTRANET(config)#interface range fa0/1-24
SW1_INTRANET(config-if-range)#shutdown
SW1_INTRANET(config-if-range)#exit
```

Una vez desactivadas las interfaces desigmaos el rango de la fastEthernet 1 a la 5 como interfaces troncales las cuales estaran vinculadas a la vlan nativa de cada intranet y las encendemos 

```
SW1_INTRANET(config)#interface range fa0/1-5
SW1_INTRANET(config-if-range)#switchport mode trunk
SW1_INTRANET(config-if-range)#switchport trunk native vlan 99
SW1_INTRANET(config-if-range)#no shutdown 
SW1_INTRANET(config-if-range)#exit
```

Ahora creamos las vlans necesarias y asignamos su nombre a cada una

```
SW1_INTRANET(config)#vlan 50
SW1_INTRANET(config-vlan)#name Ing
SW1_INTRANET(config-vlan)#vlan 100
SW1_INTRANET(config-vlan)#name Tech
SW1_INTRANET(config-vlan)#vlan 99
SW1_INTRANET(config-vlan)#name Native
SW1_INTRANET(config-vlan)#exit
```

Con las vlans creadas determinamos a que rangos de interfaces le vamos a asignar la respectiva vlan, como la guia de laboratorio no especifica que interfaces corresponde a cada vlan, las asignamos de manera equitativa entre las restantes

```
SW1_INTRANET(config)#interface range fa0/6-15
SW1_INTRANET(config-if-range)#Switchport access vlan 50
SW1_INTRANET(config-if-range)#exit
SW1_INTRANET(config)#interface range fa0/16-24
SW1_INTRANET(config-if-range)#Switchport access vlan 100
SW1_INTRANET(config-if-range)#exit
SW1_INTRANET(config)#interface range fa0/1-5
SW1_INTRANET(config-if-range)#Switchport access vlan 99
SW1_INTRANET(config-if-range)#exit
```

Finalmente le asignamos la direccion IPv6 a las interfaces de la vlan 99 en cada uno de los switches con los siguientes comandos 

```
SW1_INTRANET(config)#interface vlan 99
SW1_INTRANET(config-if)#ipv6 address 2001:1200:A1:1::2/64
SW1_INTRANET(config-if)#exit
```

No olvidar que tenemos que reactivar todas las interfaces restantes del switch y guardar los cambios 

```
SW1_INTRANET(config)#interface range fa0/6-24
SW1_INTRANET(config-if-range)#no shutdown
SW1_INTRANET(config-if-range)#exit
SW1_INTRANET(config)#exit
SW1_INTRANET#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
```

**Routers:**

Para la configuración de las VLAN, el primer paso es encender la interfaz conectada a la intranet.
```
R1_Bog(config)#interface fa0/1
R1_Bog(config-if)#no shutdown
R1_Bog(config-if)#exit
```

Ahora vamos a crear y a configurar las subinterfaces en los routers que conectan con la intranet, en cada router configuramos una subinterfaz por cada una de las vlans necesarias, a estas se les asigan la encapsulación, recordar que la vlan 99 en el caso de Bogotá y la 1 en el caso de Madrid se debe especificar que son nativas

```
R1_Bog(config)#interface fa0/1.50
R1_Bog(config-if)#encapsulation dot1q 50
R1_Bog(config-if)#ipv6 address 2001:1200:A1:2::1/64
R1_Bog(config-if)#interface fa0/1.100
R1_Bog(config-if)#encapsulation dot1q 100
R1_Bog(config-if)#ipv6 address 2001:1200:A1:3::1/64
R1_Bog(config-if)#interface fa0/1.99
R1_Bog(config-if)#encapsulation dot1q 99 native
R1_Bog(config-if)#ipv6 address 2001:1200:A1:1::1/64
```

**Router como proveedor DHCP:**

Para las dos intranets de nuestra re empresarial, utilizamos un protocolo de direccionamiento dinamico para que los host obtengan su dirección IPv6, para este laboratorio utilizamos el protocolo SLACC con DHCPv6, para esto primero necesitamos configurar una pool en el router la cual tiene almacenada la dirección IPv6 del servidor DNS

```
R1_Bog(config)#ipv6 dhcp pool bogota
R1_Bog(config-dhcpv6)#address prefix 2001:1200:A1::/48 48
R1_Bog(config-dhcpv6)#dns-server 2001:1200:C1:1::3
```

Con la pool creada lo unico que tenmos que hace es entrar a cada una de las interfaces fastEthernet que estan en uso en el router, tanto las interfaces globales, como a las subinterfaces, y una vez dentro de las interfaces habilitamos que estas puedan acceder a la pool creada, ademas ingresamos un comando para modificar el valor de la other flag que envia el router, valor que tiene que estar en 1 para que los host sepan que el protocolo que deben usar es el ya mencionado 

```
R1_Bog(config)#inteface fa0/1
R1_Bog(config-if)#ipv6 dhcp server bogota
R1_Bog(config-if)#ipv6 nd other-config-flag
R1_Bog(config-if)#exit
R1_Bog(config)#inteface fa0/1.50
R1_Bog(config-if)#ipv6 dhcp server bogota
R1_Bog(config-if)#ipv6 nd other-config-flag
R1_Bog(config-if)#exit
R1_Bog(config)#inteface fa0/1.100
R1_Bog(config-if)#ipv6 dhcp server bogota
R1_Bog(config-if)#ipv6 nd other-config-flag
R1_Bog(config-if)#exit
R1_Bog(config)#inteface fa0/1.99
R1_Bog(config-if)#ipv6 dhcp server bogota
R1_Bog(config-if)#ipv6 nd other-config-flag
R1_Bog(config-if)#exit
```

Con estos comandos es suficiente para que los host puedan obtener su dirección IPV6 por medio de SLACC usando su dirección MAC para generar automaticamente su identificador de interfaz y puede acceder al servicio DHCP v6 para obtener la información sobre el servidor DNS al que debe acceder 

## Multilayer switch 

Para la intranet de Madrid se encesita que esta cuente con un multilayer switch 3650 que haga la funcion de un switch 2810, por lo que recibe las mismas configuraciones basicas que los switches, ademas de que dentro de este tamien se encontrara la creacion y configuración de las vlans 

```
MLSW(config)#VLAN 25
MLSW(config-vlan)#name Tesoreria
MLSW(config-vlan)#vlan 50
MLSW(config-vlan)#name Vice
MLSW(config-vlan)#vlan 1
```
a la vlan 1 no se le asigno el nombre ya que este no puede ser modificado

Luego de esto configuramos el enlace troncal en el rango de interfaces de la 1 a la 3 donde la vlan 1 es la nativa, además asignamos la direccion ipv6 correspondiente a la interfaz de la vlan 1 

```
MLSW(config)#interface range gig 1/0/1-3
MLSW(config-if-range)# switchport mode trunk
MLSW(config-if-range)# switchport trunk native vlan 1
MLSW(config-if-range)# exit
MLSW(config)#interface VLAN 1
MLSW(config-if-range)# IPv6 address 2001:1200:B1:1::2/64
```
Finalemente agregamos la dirección del default gateway configurado en el router de España

```
MLSW(config)#ipv6 route ::/0 2001:1200:B1:1::1
```
## ACL'S (Access Control Lists)

**Requerimientos:** El Departamento de Tecnología solicitó los siguientes filtros de paquetes: Todos los hosts de la Intranet Bogotá acceden al servidor Web a través del protocolo HTTPs (puerto 443) y no por HTTP (puerto 80). Los usuarios del área de Vicepresidencia en la Intranet Madrid deben tener restringido el acceso al servidor Web por HTTPs (puerto 443) y HTTP (puerto 80). Los usuarios del Departamento de Tesorería sólo deben acceder al servidor Web por el puerto 443. Finalmente, El Departamento de Tecnología accede al servidor Web por HTTPs (puerto 443) y HTTP (puerto 80). ¿Qué servicio IPv6se debe configurar?

**Configuración:** Para el desarrollo de este laboratorio, fue necesario hacer uso de ACL’s (Listas de Control de Acceso) que soportaran IPv6, pues tanto el router R1_BOG y el R2_ESP hacen uso de este tipo de direccionamiento. 

Para el router de Bogotá, fue necesario hacer un bloqueo en la interfaz de salida FastEthernet 0/0, esto con el propósito de contar con un mayor control de tráfico además de evitar cargas innecesarias en el Router que tendrá que recibir paquetes que después tendrá que descartar en caso de que se pusiera en la interfaz de entrada. Poner el Access List en la interfaz de salida permite solo enviar los paquetes que han sido aceptados para ser reenviados.

El bloqueo para el R1-BOG se realizó únicamente para la Vlan de Ingeniería en el puerto 80 (HTTP) pues se nos explica que la Vlan de Tecnología no necesitara ningún tipo de bloqueo al poder ingresar por medio de cualquiera de los dos puertos 80 o 443.

El Access List se configuro de la siguiente manera:

```
R1_BOG(config)#ipv6 access-list (Nombre del Access List)
R1_BOG(config-ipv6-acl)#deny tcp (Dirección Identificadora de la Vlan) any eq (Puerto)
R1_BOG(config-ipv6-acl)#ipv6 permit any any
R1_BOG(config-ipv6-acl)#exit
R1_BOG(config)#interface (Interfaz)
R1_BOG(config-if)#ipv6 traffic-filter (Nombre del Access List) out/in
```

Ya con las direcciones establecidas por la guía de laboratorio y por el equipo, los comandos quedan de la siguiente manera:

```
R1_BOG(config)#ipv6 access-list NO-HTTP-BOGOTA
R1_BOG(config-ipv6-acl)#deny tcp 2001:1200:A1:2::/64 any eq 80
R1_BOG(config-ipv6-acl)#ipv6 permit any any
R1_BOG(config-ipv6-acl)#exit
R1_BOG(config)#interface Fa0/0
R1_BOG(config-if)#ipv6 traffic-filter NO-HTTP-BOGOTA out
```
Para comprobar la creación de la Access List, usamos el comando:
```
R1_BOG# show access-lists
```
![PRUEBA-ACL-BOGOTA](/image/PRUEBA-ACL-BOGOTA.png)

Para el R2_ESP, se uso la misma lógica que para el R1_BOG, estableciendo el Access List en la interfaz de salida del router. En este caso La Vlan de tesorería podría hacer uso únicamente del puerto 443 mientras que Vicepresidencia no podría hacer uso de este puerto ni tampoco del 80, por lo que ninguno de los dos protocolos debería funcionar para esta subred. Los comandos para esta Access List fueron los siguientes:

```
R2_ESP(config)#ipv6 access-list ACL-MADRID
R2_ESP(config-ipv6-acl)#deny tcp 2001:1200:B1:2::/64 any eq 80
R2_ESP(config-ipv6-acl)#deny tcp 2001:1200:B1:3::/64 any eq 80
R2_ESP(config-ipv6-acl)#deny tcp 2001:1200:B1:3::/64 any eq 443
R2_ESP(config-ipv6-acl)#ipv6 permit any any
R2_ESP(config-ipv6-acl)#exit
R2_ESP(config)#interface Se0/0/0
R2_ESP(config-if)#ipv6 traffic-filter ACL-MADRID out
```

```
R2_ESP#show access-lists
```
![PRUEBA-ACL-MADRID](/image/PRUEBA-ACL-MADRID.png)

Para ambos casos se añadió al final de las condiciones el comando “ipv6 permit any any” con el propósito de permitir cualquier envió de información aparte de las limitaciones establecidas en las Access Lists.
A continuación, las comprobaciones de las Access Lists en cada una de las Vlan’s:

![R1_BOG_PC1_ACCESS-LIST](/image/R1_BOG_PC1_ACCESS-LIST.jpg)

![R1_BOG_PC2_ACCESS-LIST](/image/R1_BOG_PC2_ACCESS-LIST.jpg)

![R2_ESP_PC5_ACCESS-LIST](/image/R2_ESP_PC5_ACCESS-LIST.jpg)

![R2_ESP_PC6_ACCESS-LIST](/image/R2_ESP_PC6_ACCESS-LIST.jpg)

## Enrutamiento

En este laboratorio, el enrutamiento constaba de varias partes. Primeramente, era necesario identificar los protocolos que se nos requerían en cada uno de los routers:

![PROTOCOLOS_ENRUTAMIENTO](/image/PROTOCOLOS_ENRUTAMIENTO.jpg)

Como podemos ver, tenemos dos conexiones EIGRP que hará uso de IPv4, dos conexiones que usaran OSPF v2, una conexión que hará uso de EIGRP v6 y una ultima que hará uso de OSPF v3. Nos quedará faltando una conexión que es la que se establece entre ISP_FL y ISP_NET, para esta decidimos como grupo usar EIGRP.

Como podemos ver, hay una mezcla de bastantes protocolos de enrutamiento tanto de IPv4 como de IPv6, por ello es necesario implementar la redistribución en estos protocolos.

La redistribución se refiere al proceso en el que se comparte información entre protocolos de enrutamiento, esto se hace debido a que las métricas establecidas en cada uno de los protocolos son diferentes, por ello es necesario que esta información sea comprensible para cada uno de los protocolos y así hacer que el enrutamiento general funcione de manera correcta.

Lo primero que se debe hacer, es establecer los enrutamientos que se van a usar en cada uno de los routers con sus conexiones vecinas, por ejemplo, en el ISP_FL solo habrá conexiones EIGRP, por lo que este router solo hará uso de este protocolo. Se usan los siguientes comandos:

```
ISP_FL(config)#router eigrp 1
ISP_FL(config-router)#network 196.85.201.0 0.0.0.3
ISP_FL(config-router)#network 196.85.201.12 0.0.0.3
ISP_FL(config-router)#network 196.85.201.8 0.0.0.3
```
De esta manera, el router habrá aprendido las direcciones de sus nodos vecinos. Debemos hacer esto para todos los routers, teniendo, en cuenta que si el router requiere de aprender de un vecino que use un protocolo, este router deberá tener el protocolo configurado, por ejemplo, ISP_NET tiene dos conexiones de OSPF y una de EIGRP que es la que aprenderá de ISP_FL, por ello, será necesario configurar el router con ambos protocolos de la siguiente manera:

```
ISP_NET(config)#router eigrp 1
ISP_NET(config-router)#network 196.85.201.8 0.0.0.3
ISP_NET(config-router)#exit
ISP_NET(config)#router ospf 1
ISP_NET(config-router)#network 196.85.201.16 0.0.0.3 area 0
ISP_NET(config-router)#network 196.85.201.4 0.0.0.3 area 0
ISP_NET(config-router)# router-id 2.2.2.2
```
Podemos ver que en OSPF debemos poner dos variables o parámetros nuevos que en EIGRP no se requerían, el primero es el área que es una manera de dividir una gran red en pequeñas sub redes para mejorar el rendimiento del protocolo, para el laboratorio pondremos todos los routers en área 0, y el otro es el router-id, el cual ayuda al protocolo a identificar de manera más sencilla los paquetes y los routers.

A continuación, deberemos hacer lo mismo en todos los routers, teniendo en cuenta que los que requieran de direcciones IPv6 tendrán ciertos cambios en los comandos:

EIGRP IPv6:
```
ISP_ESP(config)#ipv6 router eigrp 101
ISP_ESP(config-rtr)#eigrp router-id 3.3.3.3
ISP_ESP(config-rtr)#no shutdown
ISP_ESP(config-rtr)#exit
ISP_ESP(config)#interface se0/0/0
ISP_ESP(config-if)#ipv6 eigrp 101
```
OSPF IPv6:

```
ISP_BOG(config)#ipv6 router ospf 51
ISP_BOG(config-rtr)#router-id 1.1.1.1
ISP_BOG(config-rtr)#no shutdown
ISP_BOG(config-rtr)#exit
ISP_BOG(config)#interface se0/0/0
ISP_BOG(config-if)#ipv6 ospf 51 area 0
```
Como Podemos ver, es necesario establecer el router-id tanto para OSPF como para EIGRP además de activar “manualmente” usando “no shutdown”. Usar este tipo de direccionamiento nos da la ventaja de no tener que ingresar manualmente las direcciones vecinas si no que al activar en las interfaces los protocolos de enrutamiento, estos asignaran las direcciones automáticamente.

Por último, será necesario configurar la redistribución, en cada router que tenga conexiones que usen diferentes protocolos, será necesario asignar este proceso:

Redistribución EIGRP de Interfaces OSPF:

```
ISP_BOG(config)#router eigrp 1
ISP_BOG(config-router)#redistribute ospf 1 metric 1000 100 255 1 1500
ISP_BOG(config-router)#exit
```
Esta parte de comandos hará que los protocolos EIGRP asignados, aprendan lo que las conexiones OSPF han aprendido gracias a los vecinos, es decir, “traduce” lo aprendido por OSPF para que EIGRP lo pueda tanto usar como compartir y así completar la tabla de enrutamiento de cada uno de los routers sin importar como estén asignadas sus interfaces con sus protocolos de enrutamiento. Podemos ver que para hacer este proceso, será necesario introducir las métricas que en este caso fueron las default que usa OSPF, proceso que no habrá que hacer cuando se haga redistribución OSPF de interfaces EIGRP:

Redistribución OSPF de Interfaces EIGRP:

```
ISP_BOG(config)#router ospf 1
ISP_BOG(config-router)#redistribute eigrp 1 metric 1 subnets
ISP_BOG(config-router)#exit
```
De esta manera los diferentes protocolos podrán aprender las rutas entre ellos, es necesario hacer este proceso en cada uno de los routers con el propósito de que todas las tablas de enrutamiento queden completas y funcionando correctamente.

![TABLA-ENRUTAMIENTO-IPv4](/image/TABLA-ENRUTAMIENTO-ISP_BOG.png)

En el pantallazo se puede ver la tabla de enrutamiento resultante, es importante mencionar que en los routers que requieran IPv6, se generara una tabla con cada tipo de direccionamiento, es decir, una con IPv4 y una con IPv6 en la que se mostrarán que hay en cada tipo de direcciones.

![TABLA-ENRUTAMIENTO-IPv6](/image/TABLA-ENRUTAMIENTO-ISP_BOG-IPV6.png)


## Tunneling 
Para finalizar con las configuraciones y poder establecer conexion entre intranets y que la intranet de madrid pueda acceder a la pagina web, es la configuración de un protocolo de migración, en este caso un tunnel que nos permitira concetar los dos espacios de redes IPv6 por medio de una red IPv4. Este tunnel sera configurado en los routers que tienen anbos protocolos funcionando en el mismo router, en este caso son los ISP de bogotá y España 

Lo primero que hacemos es habilitar la interfaz del tunnel, esto se logra unicamente accediendo a la interfaz, luego le agregamos su dirección IPv6 y la fuente y destino del tunel, para la fuente se agrega la interfaz fisica conectada al router por la que se quiere enviar la información, para el caso del destino no se pone la interfaz del puerto de llegadoa, sino que se escribe la dirección IP vinculada a dicho puerto, por ultimo para terminar con la configuración basica se asigna el tipo de tunnel deseado, en este caso el tuene el de tipo ipv6ip, el cual simboliza que la encapsulación de los archivos ipv6 los transporta por medio de una encapsulación en ipv4 y este mismo es su medio de transporte 

```
ISP_BOG (config)#interface tunnel 0 
ISP_BOG(config-if)#ipv6 address 2001:1200:A1:A::1/64
ISP_BOG(config-if)#tunnel source Serial0/1/0
ISP_BOG(config-if)#tunnel destination 196.85.201.13
ISP_BOG(config-if)#tunnel mode ipv6ip
```

Aicionalmente los tuneles tenian que ser capaces de reconocer y enrutar los protocolos que hacen parte de su respectivo rouer, en el caso de Bogotá OSPFv3 y en el caos de España EIGRPv6

```
ISP_BOG(config-if)# ipv6 ospf 51 area 0
```
Finalmente tnemos que enrutar los paquetes que vengan de la red ipv6 para que puedan ir atravez del internet, para eso el enrutamiento lo dejamos dinamico de manera que si llega un paquete que se dirige a la otra intranet este se envia atravez del tunnel 

```
ISP_BOG(config)# ipv6 route 2001:1200:B1::/48 2001:1200:A1:A::2
```
Como los tuneles tienen una direccion IPv6 que hace parte del subneteo de la intranet de bogota para configurar el enrutamiento de madrid a esta especificamos que este camino los tomen las direcciones que van hacia las vlans de Bogotá

```
ISP_BOG(config)# ipv6 route 2001:1200:A1:1::/64 2001:1200:A1:A::1
ISP_BOG(config)# ipv6 route 2001:1200:A1:2::/64 2001:1200:A1:A::1
ISP_BOG(config)# ipv6 route 2001:1200:A1:3::/64 2001:1200:A1:A::1
ISP_BOG(config)# ipv6 route 2001:1200:C1::/48 2001:1200:A1:A::1
```

Como se puede ver en la imagen las rutas quedaron configuradas de manera estatica a los respectivos destinos.

![Rutas estaticas de los tuneles](/image/Rutas_Tunneles.png)


**** 
# Verificación

## Capturas del funcionamiento
- ### **Direccionamiento dinámico**
Para demostrar el correcto funcionamiento del direccionamiento dinámico a lo largo de las 4 VLANs en las que fue configurado, presentamos a continuación una captura de un host en cada una de ellas:

**VLAN 50 (BOGOTÁ)**

![PC 1 recibiendo su información de red de forma automática](/image/Direccionamiento_1.png)

**VLAN 100 (BOGOTÁ)**

![PC 2 recibiendo su información de red de forma automática](/image/Direccionamiento_2.png)

**VLAN 25 (ESPAÑA)**

![PC 5 recibiendo su información de red de forma automática](/image/Direccionamiento_3.png)

**VLAN 100 (ESPAÑA)**

![PC 6 recibiendo su información de red de forma automática](/image/Direccionamiento_4.png)


## Análisis de tráfico
Dado que los PDU simples con los que estamos familiarizados no funcionan para IPv6, hay que tener en cuenta que realizamos este rastreo de paquetes haciendo uso del hecho de que, en modo simulación, un ping a una dirección IP nos permite ver todo el camino de los paquetes ICMPv6 enviados.

Habiendo dicho esto, a continuación vamos a realizar el análisis de tráfico en dos casos que demuestran la funcionalidad de todos los requisitos de nuestro laboratorio.

### **Ping entre PC5 (Intranet Madrid) y PC1 (Intranet Bogota):**

Este análisis de tráfico corresponde al generado entre el PC5 y el PC1 cuando se genera un mensaje de ping entre los dos. 

Para empezar, como podemos ver en la siguiente captura, el ping es exitoso.

![Resultado de ping exitoso entre pc5 y pc1](/image/Ping_PC1.png)

Vamos a analizar solo uno de los paquetes enviados como parte de este proceso ping. En la siguiente imagen se puede ver el camino entero que toma el paquete desde que sale del PC5, pasando por el internet, llegando al PC1 y siendo devuelto como respuesta hasta el PC5 una vez más.

![Camino del paquete ping](/image/PDU_ronda.png)

Para empezar, como podemos ver, el paquete se envía en primera instancia con la diercción de destino del PC1, con dirección a su default gateway dado que esta dirección no está en su misma red.

![Instancia inicial del paquete ping](/image/PDU_start.png)

A continuación, una vez llega a su default gateway se puede ver que, al ser un paquete que el router R2_ESP no conoce, se envía a su ruta estática por defecto, es decir a su ISP correspondiente.

![El paquete es reenviado hacia el ISP](/image/PDU_outbound.png)

Una vez llega al router ISP_ESP, este router identifica, gracias a su enrutamiento estático, que la dirección a la que necesita llegar al paquete se encuentra al otro lado de su tunel, por lo que encapsula al paquete en un paquete de dirección IPv4 de fuente **196.85.201.13**, es decir la dirección de la interfaz por la que el router está redireccionando el paquete, y destino **196.85.201.1**, es decir la dirección de llegada del tunel.

![El paquete es encapsulado para IPv4](/image/PDU_tunnel_in.png)

Cuando el paquete termina de pasar por el tunel, es recibido una vez más por la interfaz final del tunel, y una vez aquí el router ISP_BOG lo desencapsula para revelar el paquete original con dirección de fuente y destino IPv6 una vez más. Puesto que este router conoce, gracias al enrutamiento dinámico, que la red de destino se encuentra por ese camino, redirecciona el paquete hacia la interfaz que le conecta con la intranet del PC1.

![El paquete es desencapsulado y redireccionado](/image/PDU_tunnel_out.png)

Una vez el paquete se hace camino hasta el host de destino, este procesa el mensaje y envía una respuesta al ping, direccionandola al mismo host que le envió la petición en primer lugar.

Igual que antes, el host no está en su misma red, por lo que se redirecciona a su default gateway.

![El paquete es procesado y se envía una respuesta](/image/PDU_reply.png)

El paquete pasa por exactamente el mismo proceso por el que pasó siendo enviado, puesto que los tuneles están configurados de la misma manera en los ISP_ESP e ISP_BOG, para finalmente volver al PC1, que realizó la solicitud, y ser procesado como exitoso.

![El paquete es procesado como exitoso](/image/PDU_finish.png)

Por supuesto, vale la pena mencionar que en cada paso en el que el paquete está en el Internet, este está siendo enrutado gracias a que la dirección de destino se compara con la tabla de enrutamiento que cada router ha construído y se redirecciona correspondientemente. Podemos ver un ejemplo de esto a continuación.

![Camino del paquete ping](/image/PDU_routing.png)

Así, podemos verificar que tanto nuestro método de migración como nuestro enrutamiento y direccionamiento funcionan correctamente para permitir la comunicación entre intranets.

### **Petición TCP entre PC5 (Intranet Madrid) y el servidor web:**

Este segundo análisis corresponde a los intentos de entrar a la página mediante los diferentes puertos TCP que corresponden a cada uno de los protocolos web (HTTP y HTTPS) que se nos pide limitar.

Para este ejemplo, el PC5 intentará entrar a la página mediant HTTP y mediante HTTPS. Puesto que pertenece a la VLAN 25, es decir, la de Tesorería de Madrid, debería tener el acceso denegado por el puerto 80 (HTTP) pero no por el puerto 443 (HTTPS).

Para empezar, veamos el tráfico realizado por un intento de acceso mediante protocolo HTTPS.

![Tráfico generado por un intento HTTPS](/image/PDU2_start.png)

Como se puede ver, el intento contiene la totalidad de una petición DNS mediante la que traduce el nombre a una dirección IP, y una vez la obtiene realiza un proceso TCP el cual es exitoso de igual forma, lo que podemos asumir de ver que empieza y termina en el PC5 pasando por el Servidor Web.

Ahora, miremos más detenidamente el filtro por el que pasa al intentar salir por el R2_ESP.

![Comprobación contra ACL válida](/image/PDU2_valid.png)

Efectivamente, el paquete es revisado contra la Access List que el puerto tiene configurada, y al no chocar con ningún criterio, es permitida gracias al mensaje final dentro de la misma.

Así, verificamos que el PC5 puede entrar de forma exitosa a la página web solo usando el puerto que se le es permitido. Vamos a contrastar ahora lo que pasa si la petición TCP que usa es de tipo HTTP.

![Tráfico generado por un intento HTTP](/image/PDU3_start.png)

Como se puede ver, el tráfico es interrumpido a la mitad una vez se obtiene la dirección IP mediante el proceso DNS y se intenta salir de la red. A continuación, el router, devuleve una serie de mensajes que indican que el host no es alcanzable. Tomemos un vistazo más de cerca a lo que impide que el proceso termine viendo el paquete cuando pasa por R2_ESP.

![Comprobación contra ACL denegada](/image/PDU3_finish.png)

Como se esperaba, al momento de ser comprobado contra la Access List que el puerto de salida tiene configurada, podemos ver que se deniega el acceso a cualquier paquete originario de la VLAN de Tesorería que esté intentando usar el puerto 80, y por tanto, es rechazado

Con esto, hemos verificado de igual forma que el protocolo de filtrado de paquetes escogido funciona correctamente y de acuerdo a lo que el laboratorio nos exige.

****
# Retos y recomendaciones

En cuanto al direccionamiento de este proyecto, el único reto que enfrentamos fue la incorrecta asignación original de direcciones IPv4 para el Internet, lo que llevó a errores en el posterior enrutamiento.

Para ello, recomendamos no perder la concentración o confiarse en demasía debido a lo simple que es el enrutamiento IPv6, y revisar siempre las interfaces que están conectadas en la topología antes de empezar a realizar la tabla de enrutamiento que dictará las direcciones que debe tener cada interfaz.

Consideramos que uno de los retos que más atención nos demandó fue el tema del enrutamiento, pues si bien ya se había desarrollado algún tipo de práctica con uno de los dos, el hecho de tener que manejar dos protocolos al mismo tiempo y aparte requerir de Tunneling debido al IPv6, generó bastantes confusiones durante el desarrollo, sin embargo, tras consultar bastante, se lograron solucionar las dificultades que principalmente se habían generado por errores a la hora de ingresar el direccionamiento.

Las ACL’s solo presentaron ciertas dificultades en la Intranet de Madrid, pues sin saber por que, estas no funcionaban correctamente, se trataron de hacer modificaciones varias sin ningún resultado, hasta que finalmente funcionaron, se considera que fue por algún fallo de Cisco Packet Tracer, pues estas funcionaron después de reiniciar el programa, o si no, por algún error no descubierto a la hora de ingresar las direcciones. Sin embargo, esta situación no genero mayor problema a la hora de desarrollar el laboratorio, pues como ya se mencionó, se soluciono al reiniciar el programa.

El ultimo de los retos que se nos presentaron fue la configuración del tunnel ya que no contabamos con la experiencia ni la información suficiente para poder realizar la configuración, sin embargo con las alcaraciones realizadas por el profesor, además de la documentación utilizada para entender como funciona logramos solucionar los problemas presentados en base a las decisiones que tomamos para la configuración. 

La recomendación que podemos dar en base a este protocolo de migración es que en caso de tener solo dos redes interconectadas por la red IPv4, la configuración del enrutamiento es mucho más sencillo realizarla de manera estatica ya que la cantidad de especificaciones de rutas que se debe hacer no es alta y es más facil que realizar enrutamiento dinamico.


****
# Conclusiones
Para finalizar esta práctica, nos gustaría concluír unas cuantas cosas.

Para empezar, queremos recalcar lo sencillo que es trabajar con IPv6 respecto a trabajar con IPv4. Con esto nos referimos a cada parte del proceso, desde el subneteo, que se convierte en una tarea trivial, pasando por el direccionamiento dinámico, que es tan simple como permitir que cada host construya su propia dirección y reciba la información restante, hasta el enrutamiento, en cuyo caso solo hay que asignar los protocolos a las interfaces y dejar que hagan ellas el resto del trabajo.

A su vez, queremos recalcar lo útil que es ver en acción todos los protocolos de red en una simulación más acercada a la realidad. Ver el funcionamiento detenido de un proceso de Tunneling, por poner un ejemplo, es de gran ayuda para entender la forma en la que se empaquetan los mensajes y se procesan a lo largo de la topología hasta ser desempaquetados una vez más.

De igual manera, poder experimentar las configuraciones simultaneas de redes IPv4 con redes IPv6 nos permitio observar como trabajan estas dos en la actualidad y como, en un futuro, existira una mayor concentración de redes IPv6 que permitira el paso y la transformación de redes IPv4 a IPv6 de manera mucho más natural y sencilla 

