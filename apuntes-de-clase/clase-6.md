# Clase 6 – SimuPi + Node-RED por MQTT

> Material: Actividad 3 (SimuPi – Node-RED – MQTT en AWS, entrega 16/10), la guía rápida y el tutorial local de SimuPi. Antes, en la Actividad 2 se había usado el broker de **AWS IoT** (ver [Clase 5](clase-5.md)). Acá ese broker se reemplaza por el **broker MQTT que trae SimuPi**.

---

## Notas de la clase
- **Solo hay que configurar la IP del broker** en Node-RED (en la configuración del servidor MQTT): SimuPi ya trae el broker funcionando en el puerto **1883**, **sin usuario ni contraseña**.
- **`sensor/distance`** es el **tópico al que nos vamos a suscribir** con un nodo `mqtt in` para recibir las lecturas del sensor de distancia simulado.

---

## ¿Qué es SimuPi?
Es un **simulador de Raspberry Pi** hecho por la cátedra (repositorio `github.com/seguren/simupi`). Simula:
- **Sensores** (temperatura, humedad, distancia, consumo…) que publican lecturas **periódicas**.
- **Pines GPIO** de entrada y salida.
- Un **broker MQTT embebido** (puerto **1883**).
- Una **interfaz web y una API REST** (puerto **5000**) para ver el estado, cambiar de escenario, forzar valores o simular fallas.

Sirve para **desarrollar el flujo de Node-RED sin tener el hardware** y después pasarlo a una Raspberry Pi real ([Raspberry Pi](raspberry-pi.md)).

**Escenarios preconfigurados:** `garage`, `greenhouse` (invernadero) y `smart_home` (casa inteligente). Se cambian desde el selector de la interfaz web. Cada escenario tiene sus propios sensores y pines.

### Tópicos MQTT de SimuPi

| Sentido | Tópico | Contenido |
|---|---|---|
| SimuPi → Node-RED | `sensor/<nombre>` | Valor numérico con 2 decimales (ej. `24.53`), una lectura por intervalo |
| SimuPi → Node-RED | `gpio/in/<pin>` | `0` o `1` (estado de una entrada, ej. un botón) |
| Node-RED → SimuPi | `gpio/out/<pin>` | `0` o `1` para apagar o encender una salida (ej. ventilador, bomba) |

> **Atención:** SimuPi **no publica por MQTT el estado de las salidas** (`gpio/out`). Para mostrarlo en el dashboard se usa **el último comando enviado**. El estado real se ve en la interfaz web de SimuPi o en `/api/state`.

Para ver todo lo que llega: un `mqtt in` suscripto a **`sensor/#`** y otro a **`gpio/in/#`**, conectados a nodos **`debug`**.

---

## Arquitectura en AWS (Actividad 3)

```
 Navegador del alumno ──:5000──▶ ┌──────────── INSTANCIA A ────────────┐
                                 │ SimuPi: web/API :5000 · MQTT :1883   │
                                 └──────────────────▲───────────────────┘
                                                    │ MQTT 1883 (IP privada de A)
 Navegador del alumno ──:1880──▶ ┌──────────── INSTANCIA B ────────────┐
                                 │ Node-RED :1880 · Dashboard /dashboard │
                                 └──────────────────────────────────────┘
          (las dos t3.micro con Debian 13, en la misma VPC y subred)
```

### Grupos de seguridad: el corazón de la seguridad
- **SSH (22):** solo desde la IP del alumno.
- **Instancia A:**
	- **1883 (MQTT)** → solo desde el **grupo de seguridad de B**. Se puede usar **un grupo de seguridad como origen**: "cualquier instancia que pertenezca al grupo B".
	- **5000 (web/API de SimuPi)** → solo desde la IP del alumno.
- **Instancia B:** **1880** → solo desde la IP del alumno.
- **Nunca abrir 1883 ni 5000 a `0.0.0.0/0`**, porque el broker de SimuPi **no tiene usuario ni contraseña**: cualquiera podría leer o mandar comandos.

> **¿Por qué la IP *privada* de A?** Las dos instancias están en la misma VPC, así que se hablan por la red interna de AWS: no sale a Internet, no cambia al reiniciar y no hace falta abrir puertos al exterior.

### Puesta en marcha (guía rápida)
**Instancia A (SimuPi):**
```bash
sudo apt update && sudo apt install -y git
git clone https://github.com/seguren/simupi.git && cd simupi
chmod +x bin/simupi-linux-amd64
nohup ./bin/simupi-linux-amd64 > simupi.log 2>&1 &   # sigue corriendo aunque cierres SSH
curl http://localhost:5000/api/health                # debe responder {"status":"ok"}
ss -ltn | grep -E ':(1883|5000) '                    # verifica que esté escuchando en 1883 y 5000
hostname -I                                          # muestra la IP privada (la que va en Node-RED)
```
- `nohup … &` → corre el proceso **en segundo plano** y **no se corta al cerrar la sesión SSH** (otra opción es `tmux` o `screen`).
- Logs: `tail -f ~/simupi/simupi.log` · Detenerlo: `pkill -f simupi-linux-amd64`.

**Instancia B (Node-RED):** Docker + Node-RED como en la [Clase 5](clase-5.md) e instalar el dashboard: menú ☰ → *Manage palette* → *Install* → `@flowfuse/node-red-dashboard`.

**Conexión:** en cualquier nodo `mqtt in` o `mqtt out`, en *Server* va la **IP privada de A** y el **puerto 1883**. Cuando conecta, el nodo muestra **"connected"** en verde.

---

## Cómo se arma el flujo (ejemplo guiado: invernadero)

**Sensores (MQTT → dashboard):**
```
[mqtt in: sensor/temperature] ──▶ [gauge Temperatura]
                               └─▶ [debug]
[mqtt in: sensor/humidity]    ──▶ [gauge Humedad] + [debug]
```
- El `gauge` toma `msg.payload`, y su rango (mín./máx.) tiene que coincidir con el del sensor.

**Actuadores (dashboard → MQTT):**
```
[switch Ventilador] ──▶ [mqtt out: gpio/out/22]
[switch Bomba]      ──▶ [mqtt out: gpio/out/24]
```
- En el `switch`: **On value = "1"**, **Off value = "0"** (tipo *string*).
- Una `function` "1/0 → texto" convierte el comando en **ENCENDIDO / APAGADO** para mostrarlo en un nodo `text` (porque SimuPi no informa el estado de las salidas).

### Lógica de automatización (lo que más pesa en la actividad)
- **Umbral con histéresis:** encender el ventilador **por encima de 30 °C** y apagarlo **por debajo de 27 °C**.
	- **¿Por qué dos valores?** Con un único umbral (30 °C), si la temperatura oscila entre 29,9 y 30,1 °C, el ventilador se prende y apaga constantemente. La **histéresis** deja una "banda muerta" entre 27 y 30 °C en la que **se mantiene el estado anterior**.
	- Solo se publica **cuando el estado cambia** (no en cada lectura).
- **Alarma:** un indicador en el dashboard cuando un valor sale de su rango seguro (ej. > 35 °C).
- **Modo MANUAL / AUTOMÁTICO:** un switch del dashboard guarda el modo en una **variable de contexto de flujo** (`flow.modo`). En manual mandan los switches; en automático, la lógica.

```javascript
// Ejemplo de función con histéresis (nodo function)
var t = Number(msg.payload);
var encendido = flow.get('ventilador') || 0;
if (flow.get('modo') !== 'auto') return null;     // en modo manual no hace nada
if (t > 30 && encendido === 0) encendido = 1;
else if (t < 27 && encendido === 1) encendido = 0;
else return null;                                  // sin cambio: no publica
flow.set('ventilador', encendido);
return { topic: 'gpio/out/22', payload: String(encendido) };
```

> **Variables de contexto en Node-RED:** `context` (solo ese nodo), `flow` (todos los nodos de la misma pestaña) y `global` (todo Node-RED). Sirven para "recordar" estados entre mensajes.

### Escenario propio (Parte 3)
Mínimo **3 sensores** (con al menos 2 modos de simulación y parámetros realistas: rango, unidad y período), **2 salidas** y **1 entrada GPIO**, todos con **alias** con sentido. Se crea desde *Configuración → Guardar como escenario*. Ideas: cámara frigorífica, riego automático, tanque con bomba, estacionamiento con barrera, sala de servidores.

---

## API REST de SimuPi (útil para probar y depurar)

| Método | Ruta | Para qué |
|---|---|---|
| GET | `/api/health` | ¿Responde el servidor? |
| GET | `/api/state` | Estado completo (sensores, GPIO, escenario, fallas) |
| GET | `/api/scenarios` | Lista de escenarios |
| POST | `/api/scenarios/load/<nombre>` | Cambia de escenario en caliente |
| POST | `/api/sensor/<nombre>` | Fija un valor: `{"value": 31.5}` (útil para probar la histéresis) |
| DELETE | `/api/sensor/<nombre>` | Vuelve a la simulación normal |
| POST | `/api/failure/<sensor>` | Simula una falla: `stuck`, `offline` o `clear` |
| POST | `/api/gpio/<pin>/toggle` | Invierte una entrada y la publica por MQTT |
| GET | `/api/history/<sensor>` · `/api/export/<sensor>` | Historial (JSON o CSV) |

```bash
curl -X POST http://<IP de A>:5000/api/sensor/temperature -H "Content-Type: application/json" -d '{"value": 31.5}'
```
> Las salidas `gpio/out` **no tienen endpoint**: se manejan **solo por MQTT**.

### Al terminar
**Terminar las dos instancias** (A y B) para no gastar crédito.

---

## Preguntas para repasar
1. ¿Qué publica SimuPi y en qué tópicos? ¿Qué publica Node-RED?
2. ¿Por qué el puerto 1883 de A se abre solo para el grupo de seguridad de B?
3. ¿Por qué se usa la IP privada de A en Node-RED?
4. ¿Qué es la histéresis y qué problema evita?
5. ¿Qué diferencia hay entre las variables `context`, `flow` y `global`?
6. ¿Cómo mostrás el estado de una salida si SimuPi no lo publica?
