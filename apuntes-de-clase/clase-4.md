# Clase 4 – Actividad 1: servidor web en EC2, grupos de seguridad e IP elástica

> **Nota:** esta nota estaba vacía. La armé con el material de la **Actividad 1 – AWS EC2** (entrega 25/09), que es lo que se trabajaba en esas semanas. Si en esta clase vieron otra cosa, avisame y la ajusto.

---

## Objetivo de la actividad
Poner en práctica lo de la [Clase 2](clase-2.md) y la [Clase 3](clase-3.md):
- Lanzar instancias, conectarse por SSH e instalar un **servidor web (Apache)**.
- Entender cómo el **grupo de seguridad** habilita o bloquea cada tipo de tráfico.
- Ver qué pasa con la **IP pública** al apagar y prender una instancia, y cómo la resuelve una **IP elástica**.

### Condiciones de uso
- No dejar recursos encendidos sin usar: detener o terminar.
- **Etiquetar todo** con el nombre del grupo (ej.: `AXX-INSTANCIA-1`).
- Informe en PDF (`AXX-ACTIVIDAD-1.pdf`, máximo 15 MB), con capturas de la ventana completa donde se vea el usuario conectado, explicación de cada paso, problemas encontrados y fuentes.

---

## Parte 1 – Servidor web en una instancia

### 1. Lanzar la instancia y conectarse
- `t3.micro`, AMI **Debian 13**, tag `AXX-INSTANCIA-1`.
- Conexión: `ssh -i clave.pem admin@<IP pública>`.

### 2. Instalar Apache
```bash
sudo apt-get update         # actualiza la lista de paquetes de los repositorios
sudo apt-get upgrade -y     # actualiza los paquetes instalados
sudo apt-get install -y apache2
systemctl status apache2    # debería decir "active (running)"
```

> `apt` / `apt-get` es el **gestor de paquetes** de Debian: descarga el software de los repositorios, junto con sus dependencias, y lo instala. `-y` responde "sí" automáticamente a las preguntas.

### 3. Configurar el grupo de seguridad
| Tráfico | Protocolo / puerto | Origen pedido | Cómo se prueba |
|---|---|---|---|
| **HTTP** | TCP **80** | `0.0.0.0/0` | Abrir `http://<IP>` en el navegador |
| **SSH** | TCP **22** | Solo la IP o red de tu LAN | `ssh -i ...` |
| **ICMP** | ICMP (todo) | `0.0.0.0/0` | `ping <IP>` |

- La consigna pide **probar cada protocolo antes y después** de agregar su regla. **Antes** de la regla, el navegador o el ping **no responden** (se quedan esperando: el grupo de seguridad descarta el paquete en silencio). **Después**, responden.
- `0.0.0.0/0` significa "**cualquier dirección IPv4**". Para SSH conviene limitarlo a tu IP (`x.x.x.x/32`) por seguridad.

### 4. Página propia
El *DocumentRoot* de Apache en Debian es **`/var/www/html`**:
```bash
echo "AXX-INSTANCIA-1 - AÑO 2026 - PRÁCTICA 1" | sudo tee /var/www/html/index.html
```

### 5. Apagar y volver a prender → **cambia la IP pública**
Al **detener** y volver a **iniciar** la instancia, AWS le asigna **otra IP pública**. Si hacés `ping` a la IP vieja, **deja de responder**. Esa es la "situación encontrada" que pide describir la consigna.
- La **IP privada**, en cambio, **se mantiene**.

### 6. IP elástica
1. **Reservar** (*Allocate*) una IP elástica.
2. **Asociarla** a `AXX-INSTANCIA-1`.
3. Verificar que la web responde por la **IP elástica** y que **ya no** responde por la IP pública anterior: al asociar una IP elástica, **reemplaza** a la IP pública que tenía la instancia.
4. `ping <IP elástica>` → **0 % de pérdida de paquetes**.

---

## Parte 2 – Mover la IP elástica a otra instancia

1. Lanzar `AXX-INSTANCIA-2` (mismo tipo y AMI) con Apache y su propio `index.html` ("AXX-INSTANCIA-2 – AÑO 2026 – PRÁCTICA 1").
2. **Re-asociar la IP elástica** a la instancia 2. Con `ping -t` (Windows) o `ping` continuo (Linux/macOS) se ve que **la conmutación es casi instantánea**: como mucho se pierde algún paquete en el momento del cambio.
3. En el navegador, **la misma IP ahora muestra la página de la instancia 2**. Este es el mecanismo de **tolerancia a fallos y actualización sin cortes** que se vio en teoría.
4. **Apagar la instancia 2** → el `ping` a la IP elástica **deja de responder** (la IP sigue asociada, pero no hay nadie encendido detrás).
5. **Volver a prenderla** → el `ping` **vuelve a responder con la misma IP**. Esa es la diferencia con la IP pública común: **la IP elástica no cambia al detener o iniciar**.

### Cambiar el puerto de Apache de 80 a 8000
```bash
sudo nano /etc/apache2/ports.conf                       # Listen 80  ->  Listen 8000
sudo nano /etc/apache2/sites-enabled/000-default.conf   # <VirtualHost *:80>  ->  <VirtualHost *:8000>
sudo systemctl restart apache2
```
- **Además hay que agregar al grupo de seguridad una regla TCP 8000.** Si no, el navegador no llega aunque Apache esté escuchando. Es un error típico.
- Se accede con `http://<IP elástica>:8000`.

### Al terminar
**Liberar la IP elástica** (*Release*) y **terminar** ambas instancias. Una IP elástica reservada **se cobra aunque no esté en uso**.

---

## Conceptos clave que deja la actividad
- **Grupo de seguridad = firewall con lista blanca:** lo que no está permitido explícitamente, se descarta.
- **IP pública dinámica vs. IP elástica:** la pública cambia al detener e iniciar; la elástica es fija, pertenece a la cuenta y se puede mover entre instancias.
- **IP privada:** fija mientras exista la instancia; solo sirve dentro de la VPC.
- **Abrir un puerto requiere dos cosas:** que el **servicio escuche** en ese puerto **y** que el **grupo de seguridad lo permita**.

## Preguntas para repasar
1. ¿Por qué, antes de crear la regla, el ping "se queda colgado" en lugar de dar error?
2. ¿Qué pasa con la IP pública y con la privada al detener e iniciar una instancia?
3. ¿Qué ventaja da reasociar una IP elástica frente a cambiar un registro DNS?
4. Si cambiás Apache al puerto 8000 y no podés entrar desde el navegador, ¿qué revisás?
