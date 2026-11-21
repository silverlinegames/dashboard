# DevBoard — Pizarrón Colaborativo en Tiempo Real

Pizarrón colaborativo en vivo para el equipo de **Silverline Games**: notas
adhesivas, texto, formas, conectores, dibujo libre e imágenes sobre un lienzo
infinito — sincronizado **entre dispositivos** en tiempo real, en el mismo
dashboard.

## Flujo: primero el proyecto, luego el código de invitación

1. **Crear proyecto**: escribes el nombre del proyecto y el tuyo, y se genera
   un **código de invitación** único (ej. `k7m2-9qx4`), como si fuera un
   código de sala. Al crearlo se abre directamente el modal de invitación.
2. **Invitar al equipo**: ya dentro del proyecto, el botón **Invitar** (o el
   chip con el código en la barra superior) muestra el código y permite
   copiar el código, copiar el enlace (`?sala=<código>`) o compartirlo.
3. **Unirse con código**: quien reciba el código lo escribe en la pestaña
   **"Unirme con código"** y entra al mismo tablero en vivo.
4. **Mis proyectos**: cada dispositivo guarda la lista de proyectos en los
   que trabajaste (nombre, código y última apertura) para volver con un clic.
   El menú de opciones tiene **"Cambiar de proyecto"** para moverte entre
   ellos sin recargar.

## Se guarda todo: cada cambio de cualquier persona

- **Cada edición se difunde al instante** (crear, mover, redimensionar,
  escribir texto, borrar, limpiar, renombrar el proyecto) con QoS 1 hacia
  todos los brokers conectados.
- **Respaldo local inmediato**: cada cambio —propio o de cualquier compañero—
  se guarda en `localStorage` del dispositivo.
- **Snapshot retenido en TODOS los brokers**: el estado completo queda
  retenido en cada broker MQTT conectado, así quien entre después (incluso
  si ya nadie está conectado) recupera el tablero completo.
- **Último esfuerzo al cerrar**: si cierras la pestaña con cambios aún no
  retenidos, se publica el snapshot antes de salir.
- **Reconciliación periódica**: si un mensaje se pierde, todos los
  pizarrones vuelven a converger en menos de 20 s (resolución por
  `updatedAt`: última escritura gana).
- **Tombstones**: lo borrado y los tableros limpiados no "resucitan" al
  sincronizar con un dispositivo desactualizado.
- **Sin red**: todo queda en `localStorage` y se re-difunde solo al
  reconectar. Las pestañas del mismo dispositivo se sincronizan además por
  `BroadcastChannel`.

## Sincronización entre dispositivos (malla multi-broker)

Cada dispositivo se conecta **a la vez** a varios brokers MQTT públicos sobre
WebSocket seguro (wss) — EMQX, HiveMQ, Mosquitto y Eclipse — y se suscribe al
tema de su proyecto en cada uno. Por eso la sincronización funciona entre
redes distintas (WiFi de casa, datos móviles, oficina) y no se rompe aunque a
un dispositivo le fallen algunos brokers: dos dispositivos siempre comparten
al menos un broker vivo.

- **Sin split-brain**: no hay "rotación" a un único broker; cada broker se
  reconecta a sí mismo con backoff.
- **Fan-out con deduplicación**: si un mensaje llega por dos brokers, se
  procesa una sola vez (`uid`).
- **Presencia en vivo**: cursores con nombre y avatares; latido cada 5 s por
  todos los brokers y avisos de desconexión (Last-Will de MQTT).

## Uso

1. Abre la página.
2. **Crea un proyecto** (o únete con un código de invitación).
3. Comparte el código con el botón **Invitar**.
4. Colabora: cada cambio de cualquier persona se guarda y se sincroniza.
5. Diagnóstico rápido en la consola del navegador: `__devboardDebug.estado()`
   (muestra proyecto, brokers conectados, miembros, elementos y tombstones).

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
