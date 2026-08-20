# 🌬️ Panel de control HVAC Multi‑Tasmota IR (COOLIX) — V2

Archivo principal: [hvac_V2.html](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac_V2.html)

Panel web **HTML + JS vanilla sin dependencias** para controlar **varios equipos de aire acondicionado** simultáneamente, cada uno con su propio ESP con **Tasmota versión IR** y el comando nativo `IRHVAC` (configurado por defecto para el protocolo **COOLIX**).

> Diferencias clave respecto a la V1 (`hvac.html`):
- Gestión de **varios Tasmota** (nombre nemotécnico + IP) en lugar de 1 sola IP.
- **Selector de dispositivo ACTIVO** siempre visible arriba, sin tener que entrar en ajustes.
- **Estado HVAC INDEPENDIENTE por cada dispositivo (Temp / Modo / Vent no se mezclan entre Salón y Dormitorio).
- **Importación automática** desde la V1 (IP y estado guardados en `tasmota.hvac.*`) sin tener que reconfigurar.

---

## 1. Requisitos hardware / firmware

- **1 o varios** ESP32 / ESP8266 con **Tasmota con soporte IR** (binarios `tasmota-ir` o build propio con `#define USE_IR_REMOTE`).
- Cada Tasmota debe tener: LED IR conectado al pin GPIO correcto (`IRsend`) y configurado en Tasmota como `IRsend`.
- Equipo(s) A/A con protocolo **COOLIX** (se puede cambiar vendor en el código, ver §5).
- El/los ESP y el navegador en la misma red Wi‑Fi (o ser alcanzables por IP).

> Nota: si solo es obligatorio subir este HTML a **cada Tasmota individual (§6.1) si quieres CORS perfecto, pero puedes servirlo UNA SOLA VEZ desde tu Mac/Raspberry/etc y que esa web controle todos los Tasmota sin tener que instalar nada en cada ESP.

---

## 2. Primer uso (30 s)

1.  Sirve el fichero **`hvac_V2.html`** desde cualquier servidor estático (opción recomendada para evitar CORS en más limpio):
    ```bash
    cd "/ruta/hacia/Deposito" && python3 -m http.server 8000
    ```
    y abre **http://127.0.0.1:8000/hvac_V2.html**.
    *(Alternativa sin servidor: doble‑click en `hvac_V2.html` en el Finder, pero CORS puede ser más restrictivo.)*

2.  Arriba a la derecha: botón **➕ Añadir** → se abre automáticamente la sección ⚙️ Gestión Dispositivos.

3.  En **➕ Añadir nuevo Tasmota**:
    *   **Nombre nemotécnico**: p. ej. `🏠 Salón`
    *   **IP / Host**: p. ej. `192.168.168.228`
    → **➕ Guardar dispositivo** (puedes pulsar Intro desde cualquiera de los dos inputs).

4.  **Repite el paso 3** por cada Tasmota IR que tengas (Dormitorio, Oficina, Terraza…).

5.  Elige **🏠 Salón** en el selector superior.

6.  Pulsa **🔌 Probar conexión** y luego **✅ ON** → el split del Salón se enciende. Cambia a 🛏️ Dormitorio en el selector y pulsa ON se envía SOLO a la IP del dormitorio.

### 💡 Modo pro (dejarlos precargados en el HTML:

Si no quieres repetir la configuración manual en cada navegador nuevo, edita [hvac_V2.html L429‑L438](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac_V2.html#L429-L438) y pon:
```js
const DEFAULT_DEVICES = [
  { id: "salon",   name: "🏠 Salón",      ip: "192.168.168.228" },
  { id: "dormi",   name: "🛏️ Dormitorio", ip: "192.168.168.229" },
  { id: "oficina", name: "🏢 Oficina",    ip: "192.168.168.230" },
];
```
Se cargan automáticamente la primera vez que abras la página.

---

## 3. Diseño de la interfaz (uso diario limpio)

Solo el panel de control HVAC + selector de dispositivo **siempre visible. El resto colapsado y recuerda su estado.

| Sección | Estado por defecto | Para qué |
|---|---|---|
| 🏠 **Selector Dispositivo** (barra superior) | Siempre visible | Elegir contra qué Tasmota IR actúan TODOS los botones (y ver nombre + IP del que acabas de seleccionar). |
| 🎛️ **Control HVAC** | Siempre visible | ON / OFF, modo, temperatura, ventilador, opciones avanzadas y **📤 Enviar** (solo actúa contra el dispositivo **activo**. |
| ⚙️ **Gestión Dispositivos + CORS** | **Colapsada** | Añadir / editar / borrar Tasmota; IP rápida + Probar conexión; Forzar no-cors; Arreglar CORS; Probar STATUS. |
| 📡 **Estado & último envío · payload & log HTTP** | **Colapsada** | Pastilla de resultado, preview del `IRHVAC` que se enviará (para el dispositivo activo) y log HTTP detallado; cada línea de log lleva **prefijo** con el **nombre del Tasmota** (`[🏠 Salón] …`) para no confundirlos cuando mandas a varios en la misma sesión.

### 3.1 Selector dispositivo (siempre arriba)
En [hvac_V2.html L162‑L171](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac_V2.html#L162-L171)

- **Desplegable compacto**: `🏠 Salón → 192.168.168.228`** → elegir el dispositivo.
- **➕ Añadir** → atajo a la tabla de gestión + focar en el nombre del nuevo dispositivo.
- **⚙️ Gestionar** → atajo directo a la sección de gestión completa.

Al cambiar la selección:
✅ Se carga su estado HVAC (Temp/Modo/Vent/… guardado.
✅ Se actualiza el input de IP rápida y los chips de conexión.
✅ Todos los botones (ON/OFF, Temp, Modo, Fan, Enviar…) pasan a dirigirse **solo al Tasmota recién elegido.

### 3.2 Controles rápidos (🎛️ Control HVAC)

*(Mismos botones y en [hvac_V2.html L188‑L292](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac_V2.html#L188-L292), **actúan SOBRE EL DISPOSITIVO ACTIVO**:

- **ON / OFF**: envían inmediatamente `Power:"On"` o `Power:"Off"`.
- **Modo** (segmentado): `Cool / Heat / Dry / Fan / Auto` → selección local (pulsa `📤 Enviar para aplicar).
- **Temperatura** (± botones grandes):
  - `[ − ]` / `[ + ]` → **envían inmediatamente** y fuerzan Power On.
  - El input numérico `16–32 ºC` es edición local (envía al pulsar `+/−` o el botón grande).
- **Ventilador** (segmentado + ⏪/⏩ a los lados):
  - `⏪` y `⏩` → envían inmediatamente.
  - `Min / Low / Med / High / Max / Auto` → selección local.
- **+ Opciones avanzadas**: SwingV, SwingH, Quiet, Turbo, Econo, Light, Beep, Filter → ajuste local.
- **📤 Enviar comando AHORA**: envía el payload completo contra el dispositivo seleccionado.

---

## 4. Comando enviado a cada Tasmota

El panel usa el mismo endpoint nativo de Tasmota, **por cada dispositivo ACTIVO** (según la IP guardada):
```
http://<IP-DEL-TASMOTA-SELECCIONADO>/cm?cmnd=<URL_encode(comando)>
```

Por defecto se envía un `Backlog` para recuperar STATUS tras IRHVAC (igual que V1, pero **contra la IP del dispositivo seleccionado):
```
Backlog IRHVAC {"Vendor":"COOLIX", ...}; STATUS
```

Cada línea del **Log HTTP** añade el nombre como prefijo, así que aunque mandes comandos interleaved a Salón + Dormitorio, identifica perfectamente cual es cada petición:
```
[🏠 Salón] 🔌 test→ http://192.168.168.228/cm?cmnd=<STATUS 0>       (req)
[🛏️ Dormitorio] → http://192.168.168.229/cm?cmnd=<Backlog IRHVAC…>  (req)
```

El payload exacto (para el dispositivo activo en cada momento) se ve siempre en **📡 Estado & último envío → "Payload que se enviará"**, actualizado en tiempo real.

---

## 5. Configuración avanzada (editar el HTML)

Edita directamente el `<script>` en [hvac_V2.html](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac_V2.html).

| Concepto | Dónde | Cómo |
|---|---|---|
| **Tasmota precargados (lista) | [hvac_V2.html L429‑L438](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac_V2.html#L429-L438) | `const DEFAULT_DEVICES = [{id,name,ip}, …];`
| **Cambiar vendor/protocolo HVAC | [buildHvacPayload](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac_V2.html#L825-L856) | `Vendor: "COOLIX"` → tu vendor (ej: `"DAIKIN"`, `"MITSUBISHI…")
| Enviar `IRHVAC` SIN Backlog | [sendToTasmota](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac_V2.html#L984-L992) | `sendBacklog: false` |
| Claves de `localStorage` | Parte superior `LS_KEY_*` | [hvac_V2.html L453‑L461](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac_V2.html#L453-L461) |

---

## 6. Modos de envío HTTP y CORS

Implementado en [enviarCmndTasmota](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac_V2.html#L890-L982):

| Modo | Cuándo | ¿Lee respuesta? |
|---|---|---|
| **AUTO (por defecto)** | Forzar sin CORS desmarcado | `fetch cors`; si falla por Access‑Control, `fetch no-cors` fallback`.
| **Forzar no-cors SIEMPRE | ☑️ Checkbox marcado (recomendado tu build actual de Tasmota IR envía comandos pero no lee respuesta) | Solo no-cors. |
| **CORS forzado sin fallback | Botones internos de diagnóstico CORS | `mode: cors` |

### 6.1 Arreglar CORS (dos caminos)

**A) CORS perfecto 0 esfuerzo: origen único al File System**

Sube `hvac_V2.html` al File System de **cada Tasmota** y abre `http://<IP>/hvac_V2.html`.
Al ser el **mismo origen** que `/cm?cmnd=...` en ese ESP, CORS nunca aplica. Puedes desmarcar Forzar no‑cors y leer STATUS.

**B) 1 servidor para TODOS los Tasmota (una sola URL)**

Sirve **una sola copia del `hvac_V2.html` desde **tu Mac/Raspberry con `python3 -m http.server 8000` (o un servidor web cualquiera). Esta única página controla **todos tus Tasmota IR simultáneamente (gracias al selector. Si alguno sin CORS, marca Forzar no‑cors y listo (el comando sí llega siempre).

**C) Comandos de configuración Tasmota**

Botón **🧙 Intentar arreglar CORS en Tasmota** envía (por dispositivo activo):
```
Backlog SetOption128 1; WebLog 2; Rule0 1; Cors 1; CorsDomain ; SaveData 1
```
Luego prueba STATUS 0 por CORS y desmarca automáticamente Forzar no‑cors si sale bien.

---

## 7. Almacenamiento local (localStorage` — independiente por navegador)

Todas las claves siguen prefijo `tasmota.hvac.*` y no colisionan con la V1. De hecho la 1ª apertura **importa automáticamente desde la V1:

| Clave (nueva V2) | Contenido |
|---|---|
| `tasmota.hvac.devices.v1` | Array `[{ id, name, ip }]` de todos los Tasmota. |
| `tasmota.hvac.active_dev_id.v1` | ID del dispositivo seleccionado. |
| `tasmota.hvac.states_per_device.v1` | `{ deviceId: {Power, Mode, Temp, FanSpeed,...}`. Estado HVAC **independiente** por cada Tasmota. |
| `tasmota.hvac.force_no_cors` | `1` / `0`. Mismo flag que en V1 (comparte valor, global para todos los dispositivos). |
| `tasmota.hvac.sections.v1` | Estado abierto/cerrado de ⚙️ Gestión y 📡 Estado. |

#### Compatibilidad hacia atrás (import automágica 1ª vez):
- Si V2) Si V2 no encuentra ningún dispositivo pero **sí** la antigua `tasmota.hvac.ip`**, crea **automáticamente un primer dispositivo llamado **📍 Tasmota (importado) con esa IP. Además, si existía `tasmota.hvac.state.v1`, lo asigna como punto de partida a todos los dispositivos.

Para borrar TODO y volver fábrica: `localStorage.clear()` en consola del navegador.

---

## 8. Troubleshooting típico

- **Cambio Salón → Dormitorio y se me carga estado HVAC correcto? Comprueba que no te equivoques en el selector; el **selector superior marca el dispositivo activo y la pastilla `🎛️ Control HVAC <span id="activeDevTag">Salón → ...</span>
- **Failed to fetch pero el Tasmota SÍ recibe**: CORS incompleto. Activa **☑️ Forzar no-cors SIEMPRE (configuración) o sube el html al File System (§6.1 A).
- **A/A no reacciona**:
  1. Visita `http://<IP>/` del Tasmota concreto y abre su web UI (confirma IP.
  2. Consola Tasmota → probar manual `IRHVAC {"Vendor":"COOLIX",...}` a ver si el split reacciona (confirmar vendor correcto.
  3. Asegúrate LED IR y GPIO IR LED IRsend` y apuntando al split.
- **Olvidé nombres IP? Abre **⚙️ Gestionar → tabla Mis Tasmota → ver todos los Tasmota con nombre + ip inline editable.
- **Quiero resetear solo un dispositivo y su estado sin tocar los demás** Tabla → 🗑️ Borrar solo fila; luego re‑crearlo limpio. Solo se borra ese, el resto siguen.

---

## 9. Resumen de ficheros de este proyecto HVAC

| Fichero | Descripción |
|---|---|
| [hvac_V2.html](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac_V2.html) | **Panel V2 Multi‑Tasmota IR **(esta versión). 1 página = controla N splits. |
| hvac_V2.md | Esta documentación. |
| [hvac.html](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac.html) | Panel V1 (1 solo Tasmota. Se mantiene por compatibilidad. |
| [hvac.md](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac.md) | Documentación de la V1 (1 Tasmota). |
| [hvac_local.html](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/hvac_local.html) | Panel compacto antiguo, optimizado para subir al File System de un único Tasmota. |
| [autoexec.be](file:///Users/cayetanog/Documents/Proyectos%20personales/Deposito/autoexec.be) | Script Berry (no recomendado para esta build Tasmota IR actual porque no le compiló las APIs HTTP Berry. Ver § troubleshooting global del proyecto). |
