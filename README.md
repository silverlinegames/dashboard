# DevBoard — Pizarrón Colaborativo en Tiempo Real

Pizarrón colaborativo en vivo para el equipo de **Silverline Games**: notas
adhesivas, texto, formas, conectores, dibujo libre e imágenes sobre un lienzo
infinito — sincronizado **entre dispositivos** en tiempo real, en el mismo
dashboard.

## Sincronización entre dispositivos (malla multi-broker)

Cada dispositivo se conecta **a la vez** a varios brokers MQTT públicos sobre
WebSocket seguro (wss) — EMQX, HiveMQ, Mosquitto y Eclipse — y se suscribe al
tema de su sala en cada uno. Por eso la sincronización funciona entre redes
distintas (WiFi de casa, datos móviles, oficina) y no se rompe aunque a un
dispositivo le fallen algunos brokers: dos dispositivos siempre comparten al
menos un broker vivo.

- **Sin split-brain**: no hay "rotación" a un único broker; cada broker se
  reconecta a sí mismo con backoff. Nunca quedan dos dispositivos conectados
  a brokers distintos sin verse entre sí (la falla más común antes).
- **Fan-out con deduplicación**: el estado del pizarrón viaja con QoS 1 hacia
  todos los brokers conectados; si un mensaje llega por dos brokers, se
  procesa una sola vez (`uid`).
- **Snapshot retenido**: quien entra a la sala cuando nadie más está conectado
  recibe igualmente el último pizarrón guardado en el broker (mensaje MQTT
  retenido).
- **Tombstones**: los elementos borrados y los pizarrones limpiados no
  "resucitan" al sincronizar con un dispositivo desactualizado.
- **Reconciliación periódica**: si un mensaje se pierde, todos los pizarrones
  vuelven a converger en menos de 20 s.
- **Sin red**: todo se guarda en `localStorage` y se re-difunde solo al
  reconectar. Las pestañas del mismo dispositivo se sincronizan además por
  `BroadcastChannel`.
- **Presencia en vivo**: cursores con nombre y avatares; latido cada 5 s por
  todos los brokers y avisos de desconexión (Last-Will de MQTT).

## Salas

- El **código de sala** es la clave del tablero: comparte el enlace con el
  botón **Invitar** (copia una URL con `?sala=<código>`).
- Todos los que entren con el mismo código verán el mismo pizarrón en vivo.
- Se puede cambiar de sala desde la pantalla de entrada ("Cambiar").

## Uso

1. Abre la página.
2. Escribe tu nombre (y cambia la sala si quieres una privada).
3. Entra al tablero y colabora.
4. Diagnóstico rápido en la consola del navegador: `__devboardDebug.estado()`
   (muestra sala, brokers conectados, miembros, elementos y tombstones).

## Herramientas

Seleccionar (V) · Mano (H) · Notas (S) · Texto · Formas · Conectores ·
Pincel · Imágenes (subir, URL o pegar Ctrl+V) · Zoom · Exportar PNG / JSON ·
Atajos de teclado · Limpiar pizarrón.

## Tecnología

- HTML + JavaScript (módulo) + Tailwind CSS — un solo archivo `index.html`.
- [MQTT.js 5](https://github.com/mqttjs/MQTT.js) sobre WebSocket seguro,
  conectado en malla a 4 brokers públicos con failover por broker.
- Sin backend, sin cuentas y sin claves.

---

© 2026 Silverline Games
