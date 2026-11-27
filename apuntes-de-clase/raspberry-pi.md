# Raspberry Pi – Hardware, GPIO y Node-RED

> Material: Taller 3 (Node-RED + Raspberry Pi), guía de instalación de pigpio y Actividad 4 (sistema IoT con Node-RED y Raspberry Pi, entrega 30/10). Placa usada: **Raspberry Pi 3 Model B**.

---

## Componentes de la placa
- **SoC (*System on a Chip*):** integra **CPU, GPU y otros componentes en un único chip**. Es el **cerebro** de la placa.
- **Memoria RAM.**
- **Puertos de periféricos:** USB, HDMI, Ethernet, jack de audio de 3,5 mm.
- **Conectividad inalámbrica:** WiFi y Bluetooth.
- **Almacenamiento microSD:** ahí está el **sistema operativo**; la placa **arranca desde la microSD** (no tiene disco).
- **Interfaz GPIO:** el conector de **40 pines** para conectar sensores y actuadores.

> **Sobre WiFi y Bluetooth:** en la Pi 3, los dos **comparten el mismo chip de radio y la banda de 2,4 GHz**. Por eso pueden **interferirse** y bajar el rendimiento si se usan a la vez. No es que sea imposible usar los dos, pero conviene tenerlo en cuenta (por ejemplo, si el WiFi anda mal, desactivar el Bluetooth).

### Uso "headless" (sin monitor)
- En el laboratorio **no tienen escritorio** (*headless*): **hay que conectarse por SSH** (`ssh usuario@<IP de la Pi>`).
- **Node-RED corre en Docker** sobre la Pi, y se accede desde el navegador a `http://<IP de la Pi>:1880`.

---

## GPIO (*General Purpose Input/Output*)
**Entrada/Salida de Propósito General.** Los pines **no tienen una función predefinida**: **su comportamiento se define por software, en tiempo real**.
- **Un pin, en un momento dado, funciona como entrada o como salida, pero no como las dos a la vez.**
	- **Entrada:** lee el estado de un sensor (ej. el PIR).
	- **Salida:** maneja un actuador (ej. un LED).

### Clasificación de los pines

| Tipo | Qué hace | Cuidado |
|---|---|---|
| **De alimentación** | Dan **+5 V** y **+3,3 V** para alimentar circuitos externos | Usar con precaución: **un sensor mal conectado puede quemar la Raspberry** |
| **Tierra (GND)** | Referencia de **0 V**, necesaria para **cerrar cualquier circuito** | Todo circuito necesita su GND |
| **GPIO digitales** | Programables como E/S. Lógica de **3,3 V** (ALTO = 3,3 V, BAJO = 0 V) | **No toleran 5 V**: meterles 5 V puede dañar el SoC |
| **Con funcionalidad multiplexada** | Algunos GPIO se configuran para protocolos serie **I2C, SPI y UART**, para sensores complejos | En la materia no los usamos |

> **I2C** se lee "I cuadrado C" (*Inter-Integrated Circuit*).

### Numeración: pin físico (Board) vs. BCM
- **Pin físico (Board):** la posición en el conector (1 a 40).
- **BCM:** el número del GPIO dentro del procesador Broadcom.
- Ej.: el **pin físico 40** es el **GPIO 21**. En los nodos `pi-gpiod` de Node-RED **se elige el pin físico**, y el nodo lo traduce a BCM.

---

## Naturaleza de la interfaz: solo estados discretos
Los GPIO solo entienden **dos estados**:
- **ALTO (*High*)** = 3,3 V = 1.
- **BAJO (*Low*)** = 0 V = 0.

**Limitación crítica:** el SoC **no tiene ADC** (conversor analógico-digital) nativo, así que **no puede medir directamente señales analógicas continuas** (potenciómetros, sensor de temperatura LM35, etc.). Para eso se necesita un **circuito externo** (un ADC como el MCP3008 conectado por SPI). Tampoco tiene **DAC** (conversor digital-analógico), es decir que no puede generar un voltaje intermedio "real": para eso se usa **PWM**.

---

## PWM: simulación de salidas analógicas
**Modulación por Ancho de Pulso (*Pulse Width Modulation*).**

1. **Conmutación rápida:** el pin alterna muy rápido entre ALTO y BAJO, y genera una **tensión promedio variable**.
2. **Ciclo de trabajo (*duty cycle*):** el **porcentaje del tiempo en ALTO**.

$$V_{promedio} = Duty \times 3{,}3\,V$$

| Duty cycle | Tensión promedio | Efecto en un LED |
|---|---|---|
| 100 % | 3,3 V | Brillo máximo |
| 50 % | ~1,65 V | Brillo medio |
| 25 % | 0,825 V | Brillo bajo |
| 0 % | 0 V | Apagado |

3. **Aplicaciones típicas:** control de **brillo de LEDs**, **velocidad de motores de corriente continua** (DC) y **posición de servomotores**.

> Con **pigpio**, el PWM se genera **por hardware con DMA** (acceso directo a memoria): es estable, no parpadea y no carga la CPU.

---

## Software: Docker + pigpio + Node-RED

### pigpio (el "demonio" del hardware)
**pigpiod** es un servicio (*daemon*) que controla los GPIO con **temporización precisa por hardware**. Node-RED, que corre dentro de Docker, **se conecta a pigpiod por red**: `localhost`, **puerto 8888**.

```bash
sudo apt update
sudo apt install pigpio -y
sudo systemctl enable --now pigpiod     # lo habilita al arrancar y lo inicia ya
sudo systemctl status pigpiod           # tiene que decir "active (running)"
```
> En **Debian 13 (Trixie)** el paquete `pigpiod` **fue retirado** de los repositorios, así que hay que **compilarlo** desde el código fuente (`make` y `sudo make install`) y crear el servicio `/etc/systemd/system/pigpiod.service` a mano (ver la guía de la cátedra). **pigpio no funciona en la Raspberry Pi 5**: ahí se usa `rpi-lgpio` o `gpiozero`.

### Paletas de Node-RED
Desde *Manage palette → Install*:
- **`@flowfuse/node-red-dashboard`** → el dashboard (en `http://<IP>:1880/dashboard`).
- **`node-red-node-pi-gpiod`** → los nodos **`pi gpiod in`** (leer un pin) y **`pi gpiod out`** (escribir un pin).
	- En cada nodo, el *Server* se configura con **Host `localhost`** y **Port `8888`**.

> El sensor de **ultrasonido** (distancia, HC-SR04) **requiere una librería o nodo adicional** de Node-RED, porque hay que **medir el tiempo** de un pulso y no solo leer 0/1. **Importante:** el pin ECHO del HC-SR04 da **5 V**, así que necesita un **divisor de tensión** para bajarlo a 3,3 V antes de entrar al GPIO.

---

## Práctica del Taller 3

### Actuadores: 3 LEDs

| LED | Pin físico | BCM | Función en el taller |
|---|---|---|---|
| Rojo | **36** | GPIO 16 | Salida digital (alerta) |
| Amarillo | **38** | GPIO 20 | **Salida PWM** (regulación de brillo) |
| Verde | **40** | GPIO 21 | Salida digital (encendido/apagado) |
| GND común | **39** | GND | Referencia de tierra (0 V) |

Cada módulo LED ya trae su **resistencia limitadora** (220–330 Ω), que evita que pase demasiada corriente y se queme el LED o el pin.

**Caso 1 – Encender o apagar un LED (salida digital):**
```
[inject payload=1] ──┐
                     ├──▶ [pi gpiod out · Pin 40 · modo digital]
[inject payload=0] ──┘
```

**Caso 1b – Regular el brillo (PWM):**
```
[ui-slider 0..100, paso 5] ──▶ [pi gpiod out · Pin 38 · modo PWM (0-100)]
```
`msg.payload` = 100 → siempre encendido; 50 → brillo medio; 0 → apagado.

### Sensor: PIR (HC-SR501, detector de movimiento)

| Terminal del PIR | Pin físico | Descripción |
|---|---|---|
| VCC | 2 (o 4) | +5 V |
| GND | 6 | 0 V |
| OUT | **8** (o **11**) | Señal digital de 3,3 V (GPIO 14 o GPIO 17) |

- **Potenciómetro de retardo:** hay que girarlo **todo hacia la izquierda** para que la señal dure lo mínimo (~3 s). Al máximo, queda en ALTO **más de 3 minutos**.
- **Si el pin 8 da lecturas erráticas:** es el **UART TXD** (la consola serie del sistema), así que conviene pasar al **pin 11**.

**Caso 2 – Lectura del sensor con habilitación desde el dashboard:**
```
[ui-switch Armar/Desarmar] ──▶ [change: flow.switch = 1 / 0]

[pi gpiod in · Pin 8] ──▶ [switch: flow.switch == 1 (num)] ──▶ [debug / alerta]
```
- La **variable de flujo** `flow.switch` guarda si la alarma está **armada**. Cualquier nodo de la **misma pestaña** la puede consultar.
- **Error común en el nodo `switch`:** en el valor de comparación hay que elegir el tipo **número (123)**. Si queda el tipo `flow.`, Node-RED busca una variable llamada `flow.1`, que no existe, y **la condición nunca se cumple**.

### Función de la Actividad 4: detectar un "toque" sin repeticiones
Detecta solo el **flanco de subida** (0 → 1) e ignora los rebotes durante 500 ms (*debounce*):
```javascript
var last = context.get('last') || 0;          // último estado visto
var now = Date.now();
var lastTime = context.get('lastTime') || 0;  // cuándo fue el último toque válido

if (msg.payload === 1 && last === 0 && (now - lastTime > 500)) {
    context.set('lastTime', now);
    context.set('last', 1);
    return {payload: "Toque detectado", timestamp: now};
}
if (msg.payload === 0) {
    context.set('last', 0);
}
return null;   // return null = no envía nada
```

---

## Actividad 4 – Sistema IoT con Raspberry Pi
- Grupos de 4. Mínimo **2 sensores distintos** del kit de 37 módulos (KY-xxx), **1 actuador** (LED, buzzer, servo, relé…), todo integrado en **Node-RED** en la Pi, un **dashboard** (opcional) y una **lógica automática**. Ej.: poca luz → encender LED; temperatura alta → sonar el buzzer.
- **Lo que se evalúa es entender el flujo completo:**

```
SENSOR ──▶ PROCESAMIENTO (Node-RED) ──▶ ACCIÓN (actuador) / VISUALIZACIÓN (dashboard)
```

---

## Preguntas para repasar
1. ¿Qué es el SoC? ¿Desde dónde arranca el sistema operativo?
2. ¿Qué significa que un pin GPIO sea "de propósito general"? ¿Puede ser entrada y salida a la vez?
3. Nombrá los tipos de pines y qué cuidado tener con cada uno.
4. ¿Por qué la Raspberry no puede leer directamente un potenciómetro? ¿Cómo se resuelve?
5. ¿Qué es el duty cycle? Calculá la tensión promedio para un 40 %.
6. ¿Para qué sirve pigpiod y cómo se conecta Node-RED a él?
7. ¿Qué error típico hace que el nodo `switch` nunca deje pasar el mensaje del PIR?
