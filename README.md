# 🌬️ Paneles HVAC HTML para Tasmota IR (COOLIX y otros vendors)

Repositorio con **2 páginas HTML autosuficientes (sin dependencias)** y una imagen de tasmota para controlar aires acondicionados a través de ESP32/ESP8266 con **Tasmota IR** (comando nativo `IRHVAC`). También incluye un **firmware Tasmota modificado (`tasmota_ir.html.bin`)** que publica automáticamente los ficheros subidos al File System en la URL `/ufs/<fichero>.html`.

---

## 📦 Qué hay en este repo

| Fichero | Descripción | Cuándo usarlo |
|---|---|---|
| **[hvac_local.html](hvac_local.html)** | Panel compacto (~15–16 KB), optimizado para ocupar poco en el File System del Tasmota. | ✅ **Casos 1 y 3 (abajo)**. Subir directamente al Tasmota y abrir desde él. |
| **[hvac_V2.html](hvac_V2.html)** | Panel completo: **gestiona VARIOS Tasmota IR simultáneamente**, cada uno con un nombre nemotécnico (Salón / Dormitorio / Oficina…) y su propio estado HVAC (Temp / Modo / Vent) guardado en `localStorage`. Incluye gestión de dispositivos, diagnósticos CORS, log HTTP, etc. | ✅ **Caso 2 (abajo)**. Servirlo 1 sola vez desde tu Mac/Raspberry/Servidor Web y usarlo para controlar todos los Tasmota a la vez. |
| **[tasmota_ir.html.bin](tasmota_ir.html.bin)** | **Firmware Tasmota modificado (ESP32/ESP8266).** Incluye: <br>• `IRHVAC` completo (vendor COOLIX y muchos más) <br>• **Endpoint `/ufs/<fichero>`**: los ficheros HTML/CSS/JS subidos al File System se sirven automáticamente por HTTP (sin CORS, mismo origen). | ✅ **Caso 1 recomendado**. Flashea este `.bin` en cada Tasmota IR que tengas para servir paneles HTML directos. |

---

## 🧰 3 formas de usar los paneles

### Caso 1 (RECOMENDADO: 0 CORS, 0 dolores) — HTML subido al propio Tasmota con `/ufs/`

> Ideal si **cada Tasmota IR controla 1 split** (p.ej. un ESP por habitación).

1.  **Flashea `tasmota_ir.html.bin`** en el ESP (Tasmotizer, ESP Web Tools, OTA desde Tasmota, etc.).
2.  Configura tu WiFi + el LED IR GPIO en Tasmota y **asegúrate** de que `IRHVAC` funciona (prueba un comando manual por Consola).
3.  En la UI de Tasmota → **Tools → Manage File system** → **Choose file** → selecciona `hvac_local.html` → **Start upload**.
4.  Una vez subido, abre:
    ```
    http://<IP-DEL-TASMOTA>/ufs/hvac_local.html
    ```
5.  **Primera (y única) vez que lo uses**: edita el propio fichero `hvac_local.html` justo antes de subirlo y pon **solo una vez** la configuración (o bien la página la detecta sola si está servida por `/ufs/`).
6.  ✅ Resultado: **NUNCA** falla el CORS (mismo origen). Puedes guardar esa URL en un icono de la pantalla inicio del móvil como si fuera una app.

> 🔁 Repite los pasos 3–5 **por cada Tasmota IR** que tengas: Salón → `192.168.168.228/ufs/hvac_local.html`, Dormitorio → `192.168.168.229/ufs/hvac_local.html`, etc.

---

### Caso 2 — Panel Multi‑Tasmota `hvac_V2.html` servido desde 1 sola máquina

> Ideal si **prefieres una ÚNICA URL para controlar TODOS los splits** (no quieres ir saltando entre IPs).

1.  Sirve `hvac_V2.html` con cualquier servidor web estático (Mac, Raspberry, NAS):
    ```bash
    # Ejemplo rápido con Python 3:
    cd /ruta/hacia/esta/carpeta
    python3 -m http.server 8000
    ```
2.  Desde CUALQUIER dispositivo en tu red WiFi abre:
    ```
    http://<IP-DE-TU-MAC-RASPBERRY>:8000/hvac_V2.html
    ```
3.  La 1ª vez, arriba a la derecha pulsa **➕ Añadir** y crea tus Tasmota IR:
    | Nombre | IP |
    |---|---|
    | 🏠 Salón | `192.168.168.228` |
    | 🛏️ Dormitorio | `192.168.168.229` |
    | 🏢 Oficina | `192.168.168.230` |

4.  Si al enviar un comando te dice "CORS" (el Tasmota no envía `Access-Control-Allow-Origin`) tienes 2 soluciones:
    - **A) (más fácil)**: En ⚙️ Gestión Dispositivos → marca ☑️ **Forzar modo no-cors SIEMPRE**. El comando SÍ llega y se ejecuta, solo no podrás leer la respuesta JSON.
    - **B) (perfecto)**: flashea `tasmota_ir.html.bin` que lleva ajustes de CORS listos, o ejecuta en cada Tasmota:
      ```
      Backlog SetOption128 1; WebLog 2; Rule0 1; Cors 1; CorsDomain ; SaveData 1
      ```

---

### Caso 3 — Panel compacto `hvac_local.html` sin `/ufs/` (firmware Tasmota genérico IR)

> Si prefieres un build Tasmota oficial IR normalito sin modificar.

1.  Sube `hvac_local.html` al File System del Tasmota: **Tools → Manage File system → Start upload**.
2.  Algunos builds de Tasmota sirven los ficheros subidos directamente por:
    ```
    http://<IP-DEL-TASMOTA>/hvac_local.html
    ```
3.  Si no te funciona, o te **descarga el fichero como attachment** en vez de renderizarlo, flashea `tasmota_ir.html.bin` y usa `/ufs/hvac_local.html` (Caso 1).

---

## ⚙️ Configuración 1 solo punto (recomendado antes de subir)

Para `hvac_local.html` y `hvac_V2.html` (la versión `hvac_spl.html` tiene lo propio) puedes editar las constantes arriba del `<script>` **una sola vez** y ya no tener que tocar nada más.

| Concepto | Dónde | Qué poner |
|---|---|---|
| **IP / Vendor `hvac_local.html`** | Cabecera `<script>` del fichero `hvac_local.html` | IP fija o vacía si está en `/ufs/` del Tasmota. |
| **Tasmota precargados en `hvac_V2.html`** | `const DEFAULT_DEVICES = [...]` en `hvac_V2.html` | Ej: `[{ id:"salon", name:"🏠 Salón", ip:"192.168.168.228" }]` → ya aparecen creados la primera vez que abres el panel, sin tener que añadirlos a mano. |
| **Vendor IRHVAC** (si tu AA NO es COOLIX) | Constante `HVAC_VENDOR` (o `VENDOR`) en cada HTML | Valores válidos: `COOLIX`, `DAIKIN`, `MITSUBISHI_HEAVY_88`, `MIDEA`, etc. (ver docs `IRHVAC` de Tasmota). |

---

## 🧩 Flujo de envío de comandos IRHVAC

Cualquiera de los 3 paneles envía al Tasmota seleccionado:

```
GET  http://<IP-DEL-TASMOTA>/cm?cmnd=URL_ENCODE(<comando>)
```

donde `<comando>` =

```
Backlog IRHVAC {
  "Vendor":"COOLIX",
  "Model":-1,
  "Command":"Control",
  "Mode":"Cool",
  "Power":"On",
  "Celsius":"On",
  "Temp":25,
  "FanSpeed":"Min",
  ...
}; STATUS
```

(`Backlog …; STATUS` para recuperar el estado si el CORS está OK.)

---

## 🔐 Nota sobre `tasmota_ir.html.bin`

-   El fichero `.bin` incluido (`tasmota_ir.html.bin`) es un build Tasmota **modificado con soporte completo IR + path de publicación `/ufs/`**.
-   Úsalo con tus ESPs de confianza (si no conoces el origen del `.bin`, puedes **replicar la build** tú mismo activando en `user_config_override.h`:
    -   `USE_IR_REMOTE`, `USE_IR_SEND_NEC`, etc. (estándar IRHVAC)
    -   Y la feature que publica el File System interno en `/ufs/<filename>` (según la modificación concreta de este build).
-   Flashea siempre de forma segura (Tasmotizer, web OTA si ya tienes un Tasmota base).
-   Esta basado en la version 15.4 estable. creado el 2026/08/20. Es facil regenerarlo con TasmoCOmpiler, para ello se añade el fichero ùser_config_override.h'. Pero recuerda elegir los siguientes paquetes: ESP32/Generic, Interfaz Web, IR completo y SD card/LittleFS como mínimo.

---

## 🧹 Troubleshooting rápido

| Síntoma | Solución |
|---|---|
| Abro `/ufs/hvac_local.html` y **no existe** o `404` | Asegúrate de haber **flasheado `tasmota_ir.html.bin`** antes; si no, prueba la URL clásica `/hvac_local.html` (Caso 3). |
| Panel dice "Failed to fetch" pero el Tasmota SÍ recibe el comando y el AA reacciona | CORS incompleto. Activa **Forzar modo no-cors SIEMPRE** o **sube el HTML al Tasmota** (Caso 1, `/ufs/`) = CORS perfecto. |
| El AA **no reacciona** de ninguna manera | 1) Prueba un comando `IRHVAC` manual por Consola Tasmota. 2) Asegúrate vendor correcto (no todos son COOLIX). 3) LED IR + GPIO bien configurados. |
| Navegador móvil "no caen" los botones +/- | Los botones tienen `min-height 52-56px` (dedo friendly). Si sigues fallando, usa Chrome/Safari actualizados. |
| `hvac_V2.html` mezcla las temperaturas de Salón y Dormitorio | No debería: cada dispositivo tiene **estado HVAC independiente** guardado en `tasmota.hvac.states_per_device.v1`. Revisa que el selector superior marque el dispositivo correcto (la etiqueta lo pinta todo: `<nombre> → <IP>`). |
