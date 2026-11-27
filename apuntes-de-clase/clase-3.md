# Clase 3 – Caso DonWeb y Taller 1: AWS EC2

> Primera parte: análisis de la caída de DonWeb (diapositivas de redundancia de la Teoría 1). Segunda parte: **Taller 1**, primera instancia en AWS EC2.

---

## Caso DonWeb – Nodo NOVA (Argentina)

DonWeb es un proveedor argentino de hosting y cloud. Una falla en su **nodo NOVA** dejó **inaccesibles** los servicios de **cientos de PyMEs durante unas 96 horas** (tiendas online, facturación electrónica, etc.).

### Arquitectura del nodo NOVA
- **Orquestador (IaaS) basado en OpenStack:** OpenStack es el **gestor de nube** que administra cómputo, red y almacenamiento de los VPS (*Cloud Servers*). El **plano de control** está separado de la lógica de aprovisionamiento del tráfico de datos.
- **Cómputo y almacenamiento desacoplados:** los hipervisores ejecutan la CPU y la memoria de las instancias de los clientes y se conectan por redes internas de alta velocidad a un **cluster de almacenamiento distribuido**.
- **Resiliencia teórica:** cada dato se guarda en **3 copias simultáneas** (**replicación 3x**) en hardware físicamente independiente. Para **fallas físicas** es un nivel de protección muy alto.

### ¿Qué salió mal? Una falla de **estado lógico** que se propagó en cascada
No fue un corte de fibra ni un disco roto (eso la replicación lo hubiera aguantado):
1. **El disparador:** un **error lógico interno en el sistema de archivos o en los metadatos** del almacenamiento, es decir, una **inconsistencia** en el acceso al storage.
2. **Efecto dominó:** como el sistema está diseñado para ser ultra-coherente, **replicó el error** en las 3 copias en muy poco tiempo.
3. **Bloqueo de backups:** el plano de control perdió la capacidad de leer los discos de los clientes y, además, **quedaron bloqueados los snapshots y backups**, porque vivían **en la misma red lógica del nodo**.

### La paradoja de la recuperación: velocidad vs. integridad
¿Arrancar rápido, con riesgo de corrupción masiva, o reconstruir lento pero seguro?
- Se **priorizó la integridad** por sobre el *uptime*: auditoría bit a bit y verificación de la consistencia de los metadatos antes de levantar el nodo.
- **Mitigación gradual:** no se podían encender miles de servidores a la vez, porque el pico de E/S (*boot storm*) volvía a colapsar los discos. Se hizo un **encendido controlado y progresivo** durante el cuarto día.
- Resultado: **0 % de pérdida definitiva de datos**, pero 4 días sin servicio.

### Para meditar (posibles preguntas de examen)
- **¿La replicación 3x reemplaza a un backup? NO.** Si el error es lógico, la replicación **copia el error en tiempo real**. **Replicación ≠ respaldo.**
- **Backups en la misma región o red:** los clientes tenían backups contratados, pero estaban **acoplados al mismo nodo**, así que no se pudieron usar → hay que aplicar la **regla 3-2-1**: **3** copias de los datos, en **2** medios distintos y **1** fuera del sitio (*offsite*).
- **SLA vs. realidad del negocio:** un SLA de infraestructura suele compensar con **créditos sobre la factura**, pero **nunca cubre el lucro cesante** (lo que el negocio deja de ganar mientras está caído).
- **Multi-cloud y *vendor lock-in*:** para tolerar la caída de un proveedor entero, se pueden usar balanceadores globales (como Cloudflare) e instancias de contingencia en otros proveedores (AWS, Azure, proveedores locales).

> **Snapshot** = copia de seguridad (instantánea) de un volumen en un momento dado. Pero un snapshot guardado en el mismo lugar que falla **no sirve como backup**.

---

## Taller 1 – AWS EC2

### Recomendaciones de uso de AWS
- Al lanzarla, la instancia **arranca a ejecutarse y a consumir crédito**. Cuando no la uses, dejala **detenida** (*stopped*) o **terminada** (*terminated*): en esos estados **no se factura el cómputo**.
	- **Ojo con la diferencia:** **detener** es como apagar la PC: se conservan el disco y la configuración. **Terminar** es **borrar** la instancia: por defecto también **se elimina el volumen EBS root** con todos sus datos. Antes de terminar, revisá que no necesites nada.
- **No modificar ni eliminar nada que no hayas creado vos** (la cuenta es compartida por todo el curso).
- **Seguir las indicaciones** para **nombrar los recursos** (tags).
- **Cada vez que se detiene y se vuelve a iniciar una instancia, cambia su IP pública**, salvo que le asignes una **IP elástica**. Funciona parecido a **reservar una IP en un DHCP**: en lugar de atar la IP a la dirección MAC, Amazon la ata a tu **cuenta** y la asociás a la instancia por su **ID**.
	- Por ej.: `102.5.9.1` ← asociada al ID de la instancia.

> Los datos de acceso a la cuenta de AWS de la cátedra no se incluyen en el repositorio.

### Pasos del taller
1. **Acceso a la consola de AWS** con la cuenta de la cátedra. La región por defecto es **EE. UU. Este (Ohio)**.
2. Ir a la **consola de EC2**.
3. **Lanzar una instancia:**
	- **AMI:** Debian 13 de 64 bits (con la leyenda "Apto para la capa gratuita").
	- **Tipo de instancia:** `t3.micro`.
	- **Almacenamiento:** cada instancia tiene **su propio disco**, que es un **volumen EBS** (almacenamiento en bloques).
	- **Grupo de seguridad:** elegir o crear uno (ver abajo).
	- **Par de claves:** crear un juego de llaves pública y privada (ver abajo).
	- **Tag:** ponerle **nombre** a la instancia, con el nombre de usuario o grupo.
4. Ubicar la instancia por su **ID** (ej.: `i-b9a3f6b2`), verificar que esté en estado **Running** y ponerle el nombre.
5. Ver la pestaña **Monitoring** (CloudWatch) y la acción **Get System Log** (log de arranque de la instancia).
6. **Conectarse por SSH.**
7. Al terminar, **terminar (o detener) la instancia**.

### Acceso: no hay consola física, se entra por SSH
- No tenemos **KVM** en el sentido de *Keyboard-Video-Mouse* (un monitor y un teclado conectados al servidor), porque la máquina está en un datacenter de Amazon. Ojo: no hay que confundirlo con el hipervisor KVM de la [Clase 1](clase-1.md).
- El acceso es **remoto**:
	- **Linux / macOS:** por **SSH** desde la terminal.
	- **Windows:** con un cliente SSH como **PuTTY**. Si la instancia **es Windows**, se entra por **RDP** (escritorio remoto).

### Pares de claves (*key pairs*)
Amazon no usa contraseña para entrar a la instancia: usa **criptografía de clave pública**.
- Al crear el par, **Amazon te da la clave privada** (archivo `.pem`). **La guardás vos y no la compartís nunca**: si la perdés, no podés entrar.
- **La clave pública** Amazon **la inserta en la VM** (en `~/.ssh/authorized_keys` del usuario).
- Al conectarte, SSH demuestra que tenés la privada que corresponde a esa pública.

```bash
chmod 400 /ruta/a/mi-clave.pem          # la clave tiene que ser de solo lectura para vos, si no ssh la rechaza
ssh -i /ruta/a/mi-clave.pem admin@<IP o DNS público>
```

> En las AMIs de **Debian el usuario es `admin`** (en Amazon Linux es `ec2-user` y en Ubuntu, `ubuntu`).
>
> **Con PuTTY (Windows):** hay que convertir el `.pem` a `.ppk` con **PuTTYGen** ("Load", "Save private key") y cargarlo en *Connection → SSH → Auth*.

### Grupos de seguridad (*security groups*)
> **Corrección:** en clase quedó anotado *"firmware"*, pero es el **firewall**. El grupo de seguridad es el **firewall virtual** de la instancia.

- Contiene las **reglas que controlan el tráfico de entrada** (y de salida) de las instancias del grupo.
- **Todo lo que no está permitido explícitamente, se bloquea.** Si querés que la instancia acepte tráfico en el puerto 80 (HTTP), tenés que **agregar una regla que lo permita** (*Allow*).
- Una instancia puede pertenecer a **varios grupos** a la vez.
- **Grupo por defecto de la cátedra:**
	- Permite **todo el tráfico de salida**.
	- Bloquea **todo el tráfico de entrada**…
	- …salvo el tráfico **entre instancias del mismo grupo**…
	- …y **SSH desde `163.10.0.0/16`** (la red de la UNLP).
- Las **reglas** se pueden cambiar en cualquier momento, aunque la instancia esté corriendo.

| Regla típica | Protocolo / puerto | Origen |
|---|---|---|
| SSH | TCP 22 | Solo tu IP (o la red de tu LAN) |
| HTTP | TCP 80 | `0.0.0.0/0` (todo Internet) |
| Ping | ICMP | `0.0.0.0/0` |

### Propiedades de una instancia: nombres e IPs
Cada instancia recibe:
- **1 nombre DNS y 1 IP públicos** → para acceder **desde Internet**. Ejemplo: `ec2-54-186-8-36.us-west-2.compute.amazonaws.com` → la IP pública es `54.186.8.36`.
- **1 nombre DNS y 1 IP privados** → solo accesibles **dentro de la red de AWS** (la VPC). Ejemplo: `ip-172-31-47-73.us-west-2.compute.internal` → la IP privada es `172.31.47.73`.

### Terminar la instancia
Hay que acordarse de **terminar o detener las instancias** para no gastar crédito. Al terminar, AWS avisa que **el volumen EBS root se elimina** y se pierden sus datos.

> **Nota:** Amazon **cobra los volúmenes que no se eliminan** aunque la instancia ya esté terminada.

---

## Preguntas para repasar
1. ¿Por qué la replicación 3x de DonWeb no evitó la caída? ¿Qué diferencia hay entre replicación y backup?
2. ¿Qué dice la regla de respaldo 3-2-1?
3. ¿Qué diferencia hay entre detener y terminar una instancia? ¿Qué se sigue cobrando en cada caso?
4. ¿Cómo funciona el acceso por par de claves? ¿Qué parte guarda Amazon y cuál guardás vos?
5. ¿Qué es un grupo de seguridad y cuáles son las reglas por defecto de la cátedra?
6. ¿Por qué cambia la IP pública al reiniciar la instancia y cómo se evita?
