# Proyecto final: robot ROBOT-02

Auto robot controlado con Node-RED sobre una Raspberry Pi 3, con comunicación por MQTT (Mosquitto). Se maneja con un joystick de PS4, frena solo ante obstáculos (sensor ultrasónico), tiene un modo automático de esquive, cambia de modo con un aplauso y se habilita con una llave RFID. Un dashboard web muestra la distancia y el estado.

## Contenido

- `flujo-final.json`: flujo de Node-RED (se importa desde *Menú → Importar*).
- `img/`: capturas de cada grupo del flujo y del dashboard.

## Nota

Esta carpeta incluye solo el flujo de Node-RED y capturas como referencia. El robot también usaba programas que no están en el repositorio:

- `buzzer.js` y las melodías de `tonos/`: hacen sonar el buzzer.
- `aplauso.py`: detecta los aplausos con el micrófono.
- `rfid.py`: lee la tarjeta de la llave RFID.
- El script de la PC que publica el estado del joystick por MQTT.

Por eso, el flujo no funciona por sí solo si se importa en otra Raspberry.

Para abrirlo en Node-RED se necesitan:

- Los nodos `node-red-node-pi-gpio`, `node-red-node-pisrf` y `@flowfuse/node-red-dashboard`.
- Un broker MQTT (Mosquitto) en `localhost:1883`.
