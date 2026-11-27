# Respuestas de repaso

> Respuestas desarrolladas de las preguntas que aparecen al final de cada apunte. La idea es que primero respondas vos y después compares. Cada respuesta indica de qué nota sale.

---

## Clase 1 – Introducción a Cloud Computing
Ver [Clase 1](clase-1.md).

### 1. ¿Por qué Cloud Computing no es lo mismo que Utility Computing?
**Utility Computing** es un **modelo de negocios**: proveer recursos de cómputo como un servicio público (como la luz o el agua), con **aprovisionamiento bajo demanda** y **pago por uso**. **Cloud Computing** nace de ese modelo, pero es **más amplio**: es un **paradigma de cómputo distribuido** que permite presentar un modelo de servicios (IaaS, PaaS, SaaS) y desplegar arquitecturas **sin que necesariamente haya un negocio detrás**.
- **Ejemplo:** un cloud privado en una universidad (con OpenNebula) es Cloud Computing (recursos virtuales, autoservicio, elasticidad), pero **no se cobra**: no es Utility Computing.
- **En síntesis:** todo Utility Computing puede montarse sobre cloud, pero no todo cloud es Utility Computing.

### 2. ¿Qué problema resuelve el aprovisionamiento bajo demanda? Da un ejemplo.
Resuelve el problema de **dimensionar la capacidad** cuando la demanda **varía mucho**:
- Si comprás servidores para el **pico**, el resto del tiempo están **ociosos** (inversión desperdiciada).
- Si los comprás para el **promedio**, en el pico el servicio **se satura o se cae**.

Con el aprovisionamiento bajo demanda **se agregan recursos cuando hacen falta y se liberan después**, pagando solo lo consumido.
- **Ejemplo:** Mercado Libre en *Hot Sale* o *Black Friday*: durante esos días necesita muchas más instancias y al terminar las devuelve.

### 3. Compará IaaS, PaaS y SaaS: qué administra el usuario y un ejemplo de cada uno.

| | Qué entrega el proveedor | Qué administra el usuario | Ejemplo |
|---|---|---|---|
| **IaaS** | Infraestructura virtual: VMs, almacenamiento, red | **SO, software instalado, configuración, datos** | **Amazon EC2** |
| **PaaS** | Entorno completo para desarrollar y desplegar (SO, *runtime*, herramientas) | **Su código o aplicación** y sus datos | Google App Engine, Heroku |
| **SaaS** | La aplicación terminada, que se usa desde el navegador | **Solo el uso y sus datos** | Gmail, Google Apps, Salesforce |

**Fundamento:** son **capas**, y cada una se apoya en la de abajo. **Cuanto más arriba, menos control y menos responsabilidad** para el usuario: en IaaS sos responsable de actualizar el SO; en SaaS no instalás ni mantenés nada.

### 4. ¿Qué es *cloud bursting* y en qué modelo de despliegue aparece?
Es cuando un **cloud privado "desborda" hacia un cloud público** para atender **picos de demanda** que superan su capacidad física. Aparece en el **cloud híbrido**.
- **Por qué existe:** la principal desventaja del cloud privado es la **escalabilidad limitada** por el hardware propio. El bursting lo compensa: la operación normal queda en el privado (eficiencia y seguridad) y solo el excedente va al público (escalabilidad).

### 5. Ventajas y desventajas del cloud público frente al privado.

| | **Cloud público** | **Cloud privado** |
|---|---|---|
| **Ventajas** | Acceso **inmediato** a muchos recursos con **inversión mínima**; escalabilidad prácticamente ilimitada; no mantenés un datacenter | **Seguridad** de datos sensibles (detrás del firewall, con las políticas propias); **eficiencia** del hardware gracias a la consolidación por virtualización; control total |
| **Desventajas** | **Costo acumulado alto** a largo plazo; dudas sobre **privacidad y seguridad**; dependencia del proveedor (*lock-in*); aspectos legales | **Escalabilidad limitada** a la capacidad física; inversión inicial y mantenimiento propios |

### 6. Diferencia entre escalabilidad y elasticidad.
- **Escalabilidad:** la **capacidad** de un sistema de **crecer** (agregar recursos) para atender más carga.
- **Elasticidad:** la capacidad de **agregar y liberar recursos dinámicamente** (y, en lo posible, de forma automática) **siguiendo la variación de la carga**.
- **La clave** está en el "**y achicarse**": un sistema puede ser escalable sin ser elástico (crece, pero los recursos quedan asignados para siempre). La elasticidad es la que permite **pagar solo por lo usado**. En AWS, se logra con **CloudWatch + Auto Scaling**.

### 7. Hipervisor tipo 1 vs. tipo 2 (con ejemplos).
- **Tipo 1 (nativo o *bare metal*):** se ejecuta **directamente sobre el hardware**, sin un SO de por medio, y funciona como un micro-kernel. Es **más eficiente**, y es el que se usa en los datacenters y los clouds. Ejemplos: **Xen, VMware ESX/ESXi, Microsoft Hyper-V**.
- **Tipo 2 (*hosted*):** se ejecuta **sobre un SO anfitrión**, como una capa intermedia entre el SO y las VMs. Es **menos eficiente** (cada operación atraviesa el SO anfitrión) y se usa en escritorio o para pruebas. Ejemplos: **VMware Workstation, VirtualBox**.
- **Matiz:** la cátedra pone a **KVM** como tipo 2, pero KVM es un módulo que **convierte al kernel de Linux en hipervisor**, y por eso muchos autores lo clasifican como tipo 1.

### 8. ¿Por qué los anillos de privilegio de x86 complican la virtualización? ¿Cómo lo resuelve cada técnica?
x86 tiene **4 anillos** (0 = kernel, el más privilegiado; 3 = aplicaciones). Los SO están hechos para **correr en el anillo 0** y ejecutar instrucciones privilegiadas (memoria, E/S). Pero si hay un hipervisor, **el anillo 0 tiene que ser del VMM**, que es el que controla el hardware y las VMs. Entonces el **SO guest queda en un anillo menos privilegiado**, y cuando intenta ejecutar una instrucción privilegiada, el VMM tiene que **capturarla y emularla**. El problema es que algunas instrucciones sensibles de x86 **no generan una trampa** que el VMM pueda capturar.

Cómo lo resuelve cada técnica:
- **Virtualización completa:** el VMM **abstrae todo el hardware** y captura y emula (o traduce) las instrucciones privilegiadas. **No hay que modificar el guest** y tiene mejor aislamiento, pero **rinde un poco menos**.
- **Paravirtualización:** se **modifica el SO guest** para que, en vez de intentar instrucciones privilegiadas, **le pida esas operaciones al hipervisor directamente** (*hypercalls*). **Rinde bien**, pero tiene **menor portabilidad**: no se puede hacer con SO cerrados como Windows. Ej.: Xen PV.
- **Asistida por hardware (HVM):** el **procesador** agrega un modo especial (**Intel VT-x, AMD-V**) en el que el guest corre "en su anillo 0" y el hardware **devuelve el control al VMM** automáticamente cuando hace falta. **No modifica el guest y rinde bien**, pero **necesita una CPU con soporte**.

---

## Clase 2 – AWS y EC2
Ver [Clase 2](clase-2.md).

### 1. ¿Qué diferencia hay entre una región y una zona de disponibilidad? ¿Para qué conviene usar varias zonas?
- **Región:** **ubicación geográfica** donde AWS tiene infraestructura (ej.: EE. UU. Este – Ohio). Los servicios y los precios pueden variar entre regiones, y los datos no salen de una región salvo que los transfieras.
- **Zona de disponibilidad (AZ):** uno o más **datacenters independientes dentro de una región**, con energía y red propias, unidos a las otras zonas por **fibra óptica rápida y privada**.
- **Por qué usar varias zonas:** si falla un datacenter (energía, incendio, red), la aplicación sigue funcionando en otra zona → **tolerancia a fallos**, con **baja latencia** porque las zonas están cerca entre sí. Se puede hacer **failover sin interrupciones**.
- Para tolerar la caída de una **región entera** (como la de Virginia del Norte en 2012), hay que distribuir **entre regiones**.

### 2. ¿Qué es una AMI y para qué sirve?
Una **AMI (*Amazon Machine Image*)** es una **imagen preconfigurada**, con el SO y, opcionalmente, software instalado, **a partir de la cual se lanzan las instancias**.
- Sirve para:
	- **Arrancar rápido** una instancia con el SO elegido (ej.: Debian 13).
	- **Clonar configuraciones:** configurás una instancia una vez, la guardás como AMI y lanzás **muchas instancias idénticas bajo demanda**. Es la base del escalado horizontal.
	- **Distribuir software:** hay AMIs gratuitas y pagas en el **AWS Marketplace**, y cualquiera puede publicar las suyas.
- Las AMIs se guardan en **S3**.

### 3. Explicá los créditos de CPU de las T2/T3 con el ejemplo de la t2.small. ¿Qué cambia entre standard y unlimited?
Las T2 y T3 son **instancias con ráfagas**: tienen un **rendimiento base** y pueden superarlo temporalmente gastando **créditos**.
- **1 crédito = 1 vCPU al 100 % durante 1 minuto.**
- **t2.small:** gana **12 créditos por hora**, que equivalen a un **20 % de base** (12 min de 60 = 20 %). Si usa menos del 20 %, **acumula** créditos (hasta un **máximo de 288**). Si necesita más del 20 %, **gasta del saldo**.
- **Modo standard** (el **default de T2**): cuando el saldo llega a cero, la instancia **queda limitada a su rendimiento base**.
- **Modo unlimited** (el **default de T3**): puede seguir haciendo ráfagas **aun con el saldo en cero**. Si el **promedio de CPU en 24 h** no supera la base, no se paga extra. Si la supera (ej.: **25 %** sostenido en una t2.small, 5 puntos arriba de su base), se cobra **U$S 0,05 por hora de vCPU** adicional.
- **Por qué existe este modelo:** muchas cargas (servidores web chicos, entornos de desarrollo) están **casi siempre ociosas** y tienen picos ocasionales. Pagar una CPU completa todo el tiempo sería un desperdicio.

### 4. Compará EBS, S3 y EFS.

| | **EBS** | **S3** | **EFS** |
|---|---|---|---|
| Tipo | **Bloques** (volumen) | **Objetos** | **Archivos** (NFS) |
| Analogía | Disco rígido | "Drive" con URL por archivo | Carpeta compartida en red |
| Acceso | Se monta como dispositivo en **una** instancia de **la misma AZ** | Por **URL / API / web**, desde cualquier lugar | Se monta por **NFS** desde **varias** instancias de la misma VPC |
| Persistencia | Independiente de la instancia | Durabilidad del 99,999999999 % | Persistente y elástico |
| Uso típico | Disco del SO, bases de datos | Backups, snapshots, AMIs, sitios web estáticos, archivos | Datos compartidos entre varias instancias |
| Costo por GB | Medio (pagás **todo el volumen reservado**) | **El más barato** | El más caro (pagás lo usado) |

### 5. ¿Por qué los snapshots son incrementales y dónde se guardan?
- Se guardan en **S3**, y se replican entre zonas de disponibilidad.
- Son **incrementales**: el primer snapshot copia todos los bloques del volumen, y los siguientes **solo guardan los bloques que cambiaron**, referenciando a los anteriores para el resto. Ej. de la diapositiva: 10 GiB → 4 GiB cambiados → 2 GiB agregados = **16 GiB guardados en total**. Con copias completas serían 32 GiB (10 + 10 + 12), porque cada snapshot copiaría el volumen entero.
- **Por qué:** **ahorra espacio y dinero** (S3 se paga por GB) y **tiempo** (cada snapshot copia menos). Aun así, cualquier snapshot permite **restaurar el volumen completo**.

### 6. ¿Qué diferencia hay entre durabilidad y disponibilidad en S3?
- **Durabilidad (99,999999999 %):** probabilidad de que **un objeto no se pierda nunca**. Se logra **replicándolo en varios dispositivos y datacenters**.
- **Disponibilidad (99,99 % anual):** probabilidad de que **puedas acceder** al objeto **en un momento dado**.
- **Un ejemplo para distinguirlas:** en la caída de S3 de 2017, **los datos no se perdieron** (la durabilidad se mantuvo), pero durante horas **no se pudo acceder a ellos** (falló la disponibilidad).

### 7. ¿Qué funciones cumple CloudWatch? ¿Cómo se relaciona con la elasticidad?
**CloudWatch** es el servicio de **monitoreo** de AWS. Tiene cuatro funciones:
1. **Recopilación:** métricas (CPU, red, disco…) y *logs* de EC2, EBS, ELB, RDS y de las aplicaciones.
2. **Monitoreo:** paneles unificados y **alarmas**.
3. **Acción:** las alarmas disparan **acciones automáticas**, como **Auto Scaling**.
4. **Análisis:** estadísticas en tiempo real e históricas (hasta 15 meses).

**Relación con la elasticidad:** la elasticidad necesita **medir para decidir**. CloudWatch detecta la carga (ej.: CPU promedio > 70 %) y, con una alarma, le indica a **Auto Scaling** que **agregue instancias**. Cuando la carga baja, las **quita**. Sin monitoreo no hay elasticidad automática.

### 8. ¿Qué es una IP elástica? ¿Qué pasa al detener y al terminar la instancia?
Es una **IPv4 pública fija que pertenece a la cuenta**, no a una instancia, y que se puede **asociar y reasociar** a cualquier instancia. Desde afuera de AWS resuelve a la IP pública de la instancia; desde adentro, a la privada.
- **Al detener:** la IP elástica **sigue asociada**. Cuando la instancia vuelve a arrancar, **tiene la misma IP** (a diferencia de la IP pública común, que cambia).
- **Al terminar:** la IP se **desasocia**, pero **sigue reservada en la cuenta**, y se cobra, hasta que la **liberes** (*Release*).
- **Para qué sirve:** **tolerancia a fallos y actualizaciones sin cortes**. Si una instancia falla o hay una versión nueva del servidor, se mueve la IP a otra instancia y los clientes no notan el cambio.
- **Límite:** 5 por región, porque las IPv4 son escasas.
- **Nota:** la diapositiva dice que "se libera" al detener o terminar. Ver la corrección en la [Clase 2](clase-2.md).

---

## Clase 3 – Caso DonWeb y Taller 1
Ver [Clase 3](clase-3.md).

### 1. ¿Por qué la replicación 3x de DonWeb no evitó la caída? ¿Qué diferencia hay entre replicación y backup?
Porque la falla fue **lógica**, no física: un **error en el sistema de archivos o en los metadatos** del almacenamiento. La replicación 3x está pensada para fallas **físicas** (un disco o un servidor que muere). Ante un error lógico, el sistema, diseñado para ser coherente, **replicó el error en las tres copias** casi al instante. Además, los backups estaban **en la misma red lógica del nodo** y también quedaron bloqueados.
- **Replicación:** mantiene **varias copias sincronizadas en tiempo real**. Protege la **disponibilidad** ante fallas de hardware, pero **copia también los errores** (corrupción, borrados accidentales, ransomware).
- **Backup:** es una **copia de un momento pasado**, **aislada** del sistema en producción, que permite **volver atrás** a un estado sano.
- **Conclusión:** **replicación ≠ backup**. Se necesitan las dos.

### 2. ¿Qué dice la regla de respaldo 3-2-1?
- **3** copias de los datos (la original más 2 backups).
- En **2** tipos de medio o almacenamiento distintos.
- **1** copia **fuera del sitio** (*offsite*): otra región, otro proveedor u otro edificio.
- **Fundamento con el caso DonWeb:** los clientes tenían backups, pero **todos acoplados al mismo nodo**. Con una copia *offsite* hubieran podido restaurar sus servicios en otro lado sin esperar 4 días.

### 3. ¿Qué diferencia hay entre detener y terminar una instancia? ¿Qué se sigue cobrando en cada caso?
- **Detener (*stop*):** es como **apagar** la PC. La instancia **sigue existiendo**, con su configuración y su volumen EBS. Al reiniciarla, **cambia la IP pública** (salvo que tenga una IP elástica).
	- **No se cobra** el cómputo.
	- **Sí se cobran** el **almacenamiento EBS** y las IPs elásticas.
- **Terminar (*terminate*):** **elimina la instancia definitivamente**. Por defecto **también borra el volumen EBS root** y sus datos. No se puede deshacer.
	- **No se cobra** el cómputo.
	- **Sí se cobran** los volúmenes que **no** se borraron y las IPs elásticas que sigan reservadas.

### 4. ¿Cómo funciona el acceso por par de claves? ¿Qué parte guarda Amazon y cuál guardás vos?
Se usa **criptografía asimétrica** en lugar de contraseñas:
- Al crear el par, **vos descargás la clave privada** (`.pem`) y la **guardás en secreto**: Amazon no conserva una copia, así que si la perdés, no podés entrar.
- **Amazon inserta la clave pública** en la instancia (`~/.ssh/authorized_keys` del usuario `admin`).
- Al conectarte (`ssh -i clave.pem admin@IP`), el servidor te plantea un desafío que **solo puede resolver quien tiene la clave privada** correspondiente a esa pública. Así se autentica **sin que la clave privada viaje por la red**.
- El `chmod 400` es necesario porque SSH **rechaza claves privadas que otros usuarios puedan leer**.

### 5. ¿Qué es un grupo de seguridad y cuáles son las reglas por defecto de la cátedra?
Es el **firewall virtual** de las instancias: un conjunto de **reglas que permiten tráfico** por protocolo, puerto y origen. Funciona como **lista blanca**: **todo lo que no está permitido, se bloquea**. Una instancia puede estar en varios grupos, y las reglas se pueden cambiar en caliente.

Reglas por defecto de la cátedra:
- Permite **todo el tráfico de salida**.
- **Bloquea todo el tráfico de entrada**, excepto:
	- el tráfico **entre instancias del mismo grupo**,
	- y **SSH desde `163.10.0.0/16`** (la red de la UNLP).

### 6. ¿Por qué cambia la IP pública al reiniciar la instancia y cómo se evita?
Las IPs públicas comunes son **dinámicas**: AWS las toma de un **pool compartido** y, al **detener** la instancia, **la devuelve** al pool (las IPv4 son escasas y no tiene sentido reservarlas para máquinas apagadas). Al volver a iniciarla, le asigna **otra IP del pool**.
- **Se evita con una IP elástica:** una IP fija asociada a la **cuenta**, que queda vinculada a la instancia aunque esta se detenga. Funciona como una **reserva de DHCP**, pero atada al ID de la instancia en lugar de a la MAC.

---

## Clase 4 – Actividad 1
Ver [Clase 4](clase-4.md).

### 1. ¿Por qué, antes de crear la regla, el ping "se queda colgado" en lugar de dar error?
Porque el grupo de seguridad **descarta los paquetes en silencio** (*drop*): no responde con un rechazo (*reject*) ni con un mensaje ICMP de "inalcanzable". El cliente no recibe **nada** y espera hasta que vence el tiempo (*timeout*).
- Esto es intencional: por seguridad, **no le da información a un posible atacante**. Si respondiera "rechazado", le estaría confirmando que ahí hay un equipo.

### 2. ¿Qué pasa con la IP pública y con la privada al detener e iniciar una instancia?
- **IP pública:** **cambia**, porque se devuelve al pool al detener la instancia y se asigna otra al iniciarla. Por eso, en la actividad, el ping a la IP vieja deja de responder.
- **IP privada:** **se mantiene** mientras la instancia exista, porque pertenece a la interfaz de red dentro de la VPC.
- **Con IP elástica:** la IP pública (la elástica) **también se mantiene**.

### 3. ¿Qué ventaja da reasociar una IP elástica frente a cambiar un registro DNS?
- **Es inmediato:** el cambio se aplica en segundos. En la actividad se ve con el `ping`: como mucho se pierde algún paquete.
- **Cambiar el DNS** depende del **TTL y de los cachés** de los resolvers y los clientes: durante minutos u horas, parte de los usuarios seguiría llegando al servidor viejo.
- Además, **los clientes que usan la IP directamente** no se enteran del cambio.
- Por eso es la técnica de **failover** y de **actualización sin cortes** que se vio en teoría.

### 4. Si cambiás Apache al puerto 8000 y no podés entrar desde el navegador, ¿qué revisás?
1. **Grupo de seguridad:** ¿hay una regla de entrada **TCP 8000**? Es el error más común.
2. **Apache escucha en el 8000:** `Listen 8000` en `/etc/apache2/ports.conf` y `<VirtualHost *:8000>` en el sitio habilitado.
3. **Reiniciaste el servicio:** `sudo systemctl restart apache2`, y `systemctl status apache2` sin errores.
4. **Verificación local:** `ss -ltn | grep 8000` (¿está escuchando?) y `curl localhost:8000` desde la instancia. Si funciona local pero no desde afuera, el problema es el grupo de seguridad.
5. **La URL incluye el puerto:** `http://<IP>:8000`.

Para que un puerto funcione hacen falta **dos cosas**: que el servicio escuche **y** que el firewall lo permita.

---

## Clase 5 – IoT, MQTT, AWS IoT y Node-RED
Ver [Clase 5](clase-5.md).

### 1. ¿En qué se diferencian M2M e IoT?
- **M2M:** comunicación **punto a punto** entre dos máquinas, por **redes telefónicas** tradicionales o móviles. Es un enlace cerrado y específico (ej.: una alarma que llama a la central).
- **IoT:** muchos objetos conectados por **redes IP**, que **envían sus datos a Internet o a la nube**, donde se procesan, se combinan y se integran con otros servicios. Busca formar **ambientes inteligentes**, con **captura autónoma de datos, eventos e interoperabilidad**.
- **IoT es "mucho más que M2M":** M2M conecta dos máquinas; IoT conecta **todo con todo**, a través de la nube.

### 2. Sensor vs. actuador, con un ejemplo de cada uno.
- **Sensor:** **detecta** un estímulo del medio y lo convierte en una **señal eléctrica** (magnitud física o química → eléctrica). Es una **entrada**. Ej.: el **PIR** del taller (movimiento) o un sensor de temperatura.
- **Actuador:** convierte energía **eléctrica en una acción** sobre el medio (movimiento, luz, sonido). Es una **salida**. Ej.: un **LED**, un buzzer, un motor o una bomba.
- **Un sistema IoT combina los dos:** sensor → procesamiento → actuador. Ej.: si el PIR detecta movimiento, Node-RED enciende el LED rojo.

### 3. Compará NFC, RFID y Bluetooth.

| | **NFC** | **RFID** | **Bluetooth 4.0** |
|---|---|---|---|
| Propósito | Intercambio instantáneo de datos entre dispositivos | **Identificar** un objeto (transmitir su identidad, como un número de serie) | Comunicación entre dispositivos personales (voz y datos) |
| Alcance | **Máx. 20 cm** | De cm a metros | Metros (red WPAN) |
| Velocidad / banda | Hasta **424 kbit/s** | — | Banda **ISM 2,4 GHz** |
| Ejemplos | Pagar con la tarjeta o el celular | Tags antirrobo en tiendas, control de inventario | Auriculares, relojes, sensores |

NFC es un **caso particular de RFID** de muy corto alcance, pensado para que la comunicación sea **intencional** (hay que acercar la tarjeta).

### 4. ¿Cómo funciona el modelo publicación/suscripción de MQTT? ¿Qué rol cumple el broker?
- Los **publicadores** envían mensajes a un **tópico** (ej.: `sensor/temperatura/sala1`).
- Los **suscriptores** se suscriben a los tópicos que les interesan (pueden usar comodines: `+` para un nivel y `#` para todos los niveles de abajo).
- El **broker** es el **intermediario central**: recibe todas las publicaciones y las **reenvía** a los suscriptores de cada tópico. Es el "director de tráfico".
- **Ventaja clave: desacoplamiento.** Publicadores y suscriptores **no se conocen**: solo conocen al broker. Se pueden agregar o quitar dispositivos sin tocar los demás, y un mismo dato puede llegarle a muchos consumidores. Además, el protocolo es **liviano**, ideal para dispositivos con pocos recursos.

### 5. Explicá los tres niveles de QoS y cuándo usar cada uno. ¿Cuáles soporta AWS IoT?
- **QoS 0 – "a lo sumo una vez":** se envía sin confirmación. **Puede perderse.** Es el más rápido. Para **lecturas periódicas de sensores**: si se pierde una, llega otra enseguida.
- **QoS 1 – "al menos una vez":** se reenvía hasta que hay confirmación. **Llega seguro, pero puede duplicarse.** Para comandos o eventos importantes donde un duplicado no es grave (ej.: encender una luz dos veces da el mismo resultado).
- **QoS 2 – "exactamente una vez":** usa un intercambio de 4 mensajes para garantizar **una única entrega**. Es el más lento. Para **facturación**: un duplicado cobraría dos veces y una pérdida no cobraría.
- **AWS IoT** usa MQTT 3.1.1 y **solo soporta QoS 0 y 1**.

### 6. ¿Por qué es importante que MQTT sea un estándar abierto?
Desde 2014 es **estándar OASIS**. Eso significa que **cualquier dispositivo que hable MQTT funciona con cualquier broker** que cumpla el estándar: Mosquitto local, HiveMQ, AWS IoT o el de SimuPi.
- **Evita el *vendor lock-in*:** podés desarrollar y probar en local con Mosquitto y pasar a la nube pública **sin reprogramar los dispositivos** (solo cambiás la dirección del broker y las credenciales).
- **Lo vivimos en la cursada:** en la Actividad 2 se usó el broker de **AWS IoT** y en la Actividad 3 el de **SimuPi**, con los mismos nodos `mqtt in` y `mqtt out` de Node-RED.

### 7. ¿Cuáles son los componentes de AWS IoT? ¿Qué es un Device Shadow?
**Componentes:**
1. **Broker de mensajes:** MQTT, HTTP o WebSockets.
2. **Gateway de dispositivos:** intercambio de mensajes por publicación/suscripción sobre tópicos.
3. **Autenticación y autorización:** autenticación mutua y cifrado en todas las conexiones (**SigV4** o certificados **X.509**).
4. **Registro de dispositivos:** le da una **identidad única** a cada dispositivo (*thing*) y define cómo se conecta.

Además están el **Rules Engine**, que procesa los mensajes y los envía a otros servicios de AWS, y los **Device Shadows**.

**Device Shadow:** un **documento en la nube con el estado del dispositivo** (el estado reportado y el estado deseado). Las aplicaciones leen y modifican el shadow **aunque el dispositivo esté desconectado**; cuando vuelve a conectarse, **se sincroniza** con el estado deseado. Es una forma simple de **gemelo digital**: operar con el dispositivo **como si estuviera digitalizado**.

### 8. ¿Qué es Node-RED? ¿Qué diferencia hay entre un nodo de inyección y uno de función?
**Node-RED** es una **herramienta de programación visual basada en flujos**, creada por IBM y *open source* desde 2016. Está basada en **Node.js** y guarda los flujos en **JSON**. Desde un **editor web** (puerto 1880) se conectan **nodos con cables** para integrar hardware, APIs y servicios **casi sin escribir código**. El dashboard es un **complemento** (`@flowfuse/node-red-dashboard`).
- **Nodo de inyección (*inject*):** **genera un mensaje sin necesitar una entrada** (manualmente, cada X segundos o en un horario) y lo envía al siguiente nodo. Es un **punto de partida** del flujo.
- **Nodo de función:** **tiene una entrada**, recibe un mensaje (`msg`), **hace algo con él** (filtrarlo, transformarlo, enviarlo por MQTT, mostrarlo) y, si corresponde, lo pasa al siguiente. Ej.: `function`, `switch`, `change`, `mqtt out`, `debug`.

### 9. ¿Para qué sirven el `chown 1000:1000` y el volumen `./data:/data` en el despliegue con Docker?
- **Volumen `./data:/data`:** **monta la carpeta `~/nodered/data` del host dentro del contenedor**, en `/data`. Ahí Node-RED guarda los **flujos, los nodos instalados y las credenciales**. Como quedan **en el host**, **persisten** aunque se borre o se actualice el contenedor, que es efímero. También facilita el backup (`tar` de esa carpeta).
- **`chown -R 1000:1000 data`:** adentro del contenedor, Node-RED corre con un usuario **sin privilegios** con **UID 1000**. Si la carpeta del host pertenece a root, **Node-RED no puede escribir** y fallan el guardado de flujos y la instalación de nodos. Con el `chown`, la carpeta pasa a ser de ese UID.

---

## Clase 6 – SimuPi + Node-RED por MQTT
Ver [Clase 6](clase-6.md).

### 1. ¿Qué publica SimuPi y en qué tópicos? ¿Qué publica Node-RED?
- **SimuPi publica:**
	- `sensor/<nombre>` → el valor de cada sensor (ej. `24.53`), una vez por intervalo. Ej.: `sensor/distance`, `sensor/temperature`.
	- `gpio/in/<pin>` → el estado de las entradas (`0` o `1`).
- **Node-RED publica:**
	- `gpio/out/<pin>` → comandos `0` o `1` para apagar o encender las salidas simuladas (ventilador, bomba, etc.).
- **SimuPi no publica el estado de las salidas:** Node-RED muestra el último comando que envió.

### 2. ¿Por qué el puerto 1883 de A se abre solo para el grupo de seguridad de B?
Porque el broker de SimuPi **no tiene usuario ni contraseña**. Si el 1883 estuviera abierto a `0.0.0.0/0`, **cualquiera en Internet** podría leer los datos y **enviar comandos** a los GPIO.
- Al usar **el grupo de seguridad de B como origen**, solo pueden conectarse **las instancias que pertenecen a ese grupo**, sin importar su IP. Es el **principio de mínimo privilegio**: cada puerto, abierto solo para quien lo necesita.
- **Con el mismo criterio:** el 5000 y el 1880 se abren solo para la IP del alumno, y el 22 también.

### 3. ¿Por qué se usa la IP privada de A en Node-RED?
- Las dos instancias están **en la misma VPC y subred**, así que se comunican por la **red interna de AWS**.
- La **IP privada no cambia** al detener e iniciar la instancia (la pública sí, si no hay IP elástica).
- El tráfico **no sale a Internet**: es más seguro y no hace falta exponer el 1883 hacia afuera.
- Además, los grupos de seguridad pueden filtrar por **grupo de origen**, lo que funciona con el tráfico interno.

### 4. ¿Qué es la histéresis y qué problema evita?
Es usar **dos umbrales distintos** para encender y para apagar. Ej.: el ventilador **se enciende por encima de 30 °C** y **se apaga por debajo de 27 °C**. Entre 27 y 30 °C, **mantiene el estado anterior**.
- **Problema que evita:** con **un solo umbral** (30 °C), si la temperatura oscila alrededor de ese valor (29,9 → 30,1 → 29,9…), el actuador **se prende y se apaga constantemente** (*chattering*). Eso desgasta motores y relés, genera ruido y llena de mensajes MQTT. La banda muerta de la histéresis lo elimina.

### 5. ¿Qué diferencia hay entre las variables `context`, `flow` y `global`?
Son espacios para **guardar datos entre mensajes** en Node-RED, y se diferencian por su **alcance**:
- **`context`:** visible **solo para ese nodo**. Ej.: en la función de "toque", `last` y `lastTime` recuerdan el estado anterior del sensor.
- **`flow`:** visible para **todos los nodos de la misma pestaña** (flujo). Ej.: `flow.modo` (MANUAL / AUTO) o `flow.switch` (alarma armada).
- **`global`:** visible para **todos los flujos** de esa instancia de Node-RED.
- Se usan con `.get('nombre')` y `.set('nombre', valor)`.

### 6. ¿Cómo mostrás el estado de una salida si SimuPi no lo publica?
**Reflejando el último comando enviado:** el mismo mensaje que va al `mqtt out` (`gpio/out/22` = "1") se manda **también** a una `function` que lo traduce a "ENCENDIDO" o "APAGADO", y de ahí a un nodo `text` del dashboard.
- **Limitación:** muestra lo que se **ordenó**, no lo que **realmente pasó**. Si el comando no llega, el dashboard muestra un estado incorrecto.
- **Para verificar el estado real:** la interfaz web de SimuPi o `GET /api/state` (campo `gpio_outputs`).

---

## Raspberry Pi
Ver [Raspberry Pi](raspberry-pi.md).

### 1. ¿Qué es el SoC? ¿Desde dónde arranca el sistema operativo?
- El **SoC (*System on a Chip*)** es un **único chip que integra CPU, GPU y otros componentes** (controladores, etc.). Es el **cerebro** de la placa. Integrarlo todo en un chip permite que la placa sea **chica, barata y de bajo consumo**.
- El SO **arranca desde la tarjeta microSD**: la Raspberry no tiene disco interno.

### 2. ¿Qué significa que un pin GPIO sea "de propósito general"? ¿Puede ser entrada y salida a la vez?
Significa que **no tiene una función fija de fábrica**: **se configura por software, en tiempo real**, como **entrada** (leer un sensor) o como **salida** (manejar un actuador).
- **No puede ser entrada y salida a la vez:** en un momento dado funciona como una cosa o la otra. Se puede **reconfigurar**, pero no las dos simultáneamente.

### 3. Nombrá los tipos de pines y qué cuidado tener con cada uno.
- **Alimentación (+5 V y +3,3 V):** alimentan circuitos externos. **Cuidado:** un cortocircuito o un sensor mal conectado puede **quemar la Raspberry**.
- **Tierra (GND, 0 V):** referencia para **cerrar el circuito**. Todo sensor o actuador necesita su GND conectado.
- **GPIO digitales:** E/S programables con **lógica de 3,3 V**. **Cuidado:** **no toleran 5 V**. Por ejemplo, el ECHO del sensor de ultrasonido necesita un **divisor de tensión**.
- **Multiplexados (I2C, SPI, UART):** para sensores complejos que se comunican por protocolos serie. **Cuidado:** si el pin está en uso por otra función (ej.: el **pin 8 es UART TXD**, la consola serie), puede dar **lecturas erráticas**. Por eso en el taller se pasa el PIR al pin 11.

### 4. ¿Por qué la Raspberry no puede leer directamente un potenciómetro? ¿Cómo se resuelve?
Porque los GPIO son **digitales**: solo distinguen **ALTO (3,3 V)** y **BAJO (0 V)**, y el SoC **no tiene ADC** (conversor analógico-digital). Un potenciómetro (o un LM35) entrega una **tensión continua variable**, por ejemplo 1,2 V, que la Pi no puede medir.
- **Solución:** un **ADC externo** (por ejemplo, un MCP3008 conectado por SPI) que convierte la tensión en un número digital, o bien usar sensores que ya entregan una salida digital.

### 5. ¿Qué es el duty cycle? Calculá la tensión promedio para un 40 %.
Es el **porcentaje del tiempo que la señal PWM está en ALTO** dentro de cada período. Como el pin conmuta muy rápido, el dispositivo "ve" una **tensión promedio**:

$$V_{prom} = Duty \times 3{,}3\,V = 0{,}40 \times 3{,}3\,V = 1{,}32\,V$$

- Con un LED, eso es un **brillo intermedio**, algo menos que el 50 % (1,65 V).

### 6. ¿Para qué sirve pigpiod y cómo se conecta Node-RED a él?
- **pigpiod** es un **demonio (servicio del sistema)** que controla los GPIO con **temporización precisa por hardware**, usando **DMA**. Gracias a eso genera un **PWM estable, sin parpadeo y sin cargar la CPU**.
- **Por qué hace falta:** Node-RED corre **dentro de un contenedor Docker**, aislado del hardware. En lugar de darle al contenedor acceso directo a los pines, **Node-RED le habla a pigpiod por red**.
- **Conexión:** se instala la paleta `node-red-node-pi-gpiod`, y en los nodos `pi gpiod in` y `pi gpiod out` el *Server* se configura con **Host `localhost`** y **Port `8888`**. pigpiod tiene que estar corriendo (`sudo systemctl status pigpiod` → *active (running)*).

### 7. ¿Qué error típico hace que el nodo `switch` nunca deje pasar el mensaje del PIR?
En la regla `flow.switch == 1`, el **tipo del valor de comparación** queda configurado como **`flow.`** en lugar de **número (123)**. Entonces Node-RED no compara contra el número 1, sino que **busca una variable de flujo llamada `1`** (`flow.1`). Como esa variable no existe, la condición **nunca se cumple** y el mensaje no llega al `debug`.
- **Solución:** elegir el tipo **number** en el desplegable que está junto al valor.
