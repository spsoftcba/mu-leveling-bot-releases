# MU Leveling Bot

## Primeros pasos

Al abrir el bot por primera vez aparece la ventana de activación. Ingresá tu clave y presioná **Activar**. Una vez activado, el bot recuerda la licencia y no vuelve a pedirla.

> Tené el juego abierto antes de abrir el bot.

---

## Archivos de configuración

El bot usa tres archivos separados que se crean automáticamente:

| Archivo | Qué guarda |
|---------|------------|
| `leveling_config_global.json` | Teclas, delays, timeouts, regiones de level/helper/mapa, palabras clave de ventana |
| `leveling_config_multi.json` | Perfiles de cada personaje con sus zonas y region de level |
| `leveling_default_profile.json` | Perfiles default para aplicar a personajes nuevos |

> Si venías usando una versión anterior, el bot migra automáticamente tu configuración al nuevo formato. Si al archivo le falta algún campo nuevo (helper_region, map_region, window_keywords), se agrega automáticamente con valores por defecto sin tocar el resto.

---

## Interfaz principal

### Barra superior

- **LEVEL** — muestra el nivel leído en tiempo real
- **[nombre del char]** — personaje de la ventana activa
- **MOUSE** — coordenadas X/Y del mouse, útil para capturar clicks
- **VENTANA** — selector de la ventana de MU a controlar. También aparecen los perfiles default como `[ [Default] Nombre ]` para visualizarlos
- **↺** — refresca la lista de ventanas detectadas

---

## Tab Control

### Botones

- **▶ Ejecutar Bot** — inicia el bot para la ventana seleccionada (o varias en modo multi)
- **■ Detener** — detiene todo
- **▷ Test Secuencia** — recorre las zonas para verificar que los clicks y recorridos estén bien, sin necesidad de tener el nivel correcto

### Teclas activas mientras el bot corre

| Tecla | Acción |
|-------|--------|
| **F10** | Detiene el bot |
| **F9** | Pausa / reanuda |
| **F8** | **Skip** — cancela el warp o walk actual y pasa al siguiente ciclo |

Las teclas se muestran debajo de los botones y se actualizan si las cambiás en Config.

---

## Tab Config

### Configuración general

Ajustes globales que aplican a todos los perfiles:

- **Teclas** — stop, pausa, skip, stats, mapa, helper
- **Delays** — tiempos de espera para abrir mapa, submenu, carga de mapa, etc.
- **Timeout inactividad** — si el nivel no cambia en este tiempo (segundos), el bot re-warpea a la zona actual
- **Delay entre ventanas** — tiempo de espera entre cada ventana en modo multi
- **Palabras clave ventana** — palabras separadas por coma que identifican las ventanas del juego (por defecto `MEGAMU,MU Online`). Cambialo si jugás en otro servidor con otro nombre de cliente

### Region Level (por perfil)

Define qué parte de la pantalla lee el bot para detectar el nivel.

1. Abrí el panel C en el juego (tecla configurada en "Tecla stats")
2. Posicioná el mouse en la esquina superior izquierda del texto "Level: XXX"
3. Presioná **left/top** — hace cuenta regresiva de 3s y captura la posición
4. Ajustá **width** y **height** para que el recuadro cubra solo el número de nivel
5. Usá **Ver region** para ver el recuadro superpuesto en pantalla (3 segundos)

### Region Helper

Define qué parte de la pantalla lee el bot para detectar si el auto-ataque está activo.

El ícono del helper tiene dos estados:
- **Rojo (pause)** → helper activo, personaje pegando → OK
- **Verde (play)** → helper detenido, personaje parado o muerto → el bot re-warpea

Capturá la región igual que la del level: posicioná el mouse sobre el ícono del helper y usá **left/top**. Ajustá width/height para cubrir solo el ícono (40x40 suele ser suficiente).

Usá **Test RGB** para verificar que la región está bien configurada — te muestra los valores R/G y el estado detectado en ese momento.

> Si la región no está configurada (left y top en 0), el bot omite esta verificación sin errores.

### Region Mapa

Define qué parte de la pantalla lee el bot para detectar el nombre del mapa actual (esquina superior izquierda del juego, donde aparece "Atlans", "Kanturu", etc.).

Capturá la región posicionando el mouse en la esquina superior izquierda del texto del mapa y usando **left/top**. Ajustá width/height para cubrir el nombre completo (200x40 suele alcanzar).

Usá **Test OCR** para verificar que el bot lee correctamente el nombre del mapa en ese momento.

> Si la región no está configurada, el bot omite esta verificación sin errores.

### Export / Import

- **↑ Exportar config global** — guarda teclas, delays y configuración en un archivo JSON
- **↓ Importar config global** — carga esa configuración (pide confirmación antes de pisar la actual)

---

## Tab Zonas

Lista de zonas de leveling del perfil activo.

### Lógica del bot por zona

El bot revisa tres condiciones en cada ciclo, en orden:

1. **Trigger de level** — si el personaje llegó al nivel de trigger de una zona nueva, warpea a esa zona inmediatamente
2. **Estado del helper** — si no hay trigger nuevo, verifica si el helper está activo:
   - Running (rojo) → pasa a la condición 3
   - Stopped (verde) → el personaje está parado o murió → re-warpea (con rotación de spot si agotó reintentos)
3. **Verificación de mapa** — si el helper está activo, lee el nombre del mapa actual y verifica que coincida con la zona donde debería estar el personaje. Si no coincide → re-warpea a la zona correcta

> La condición 3 es especialmente útil cuando el personaje sube rápido de nivel, llega al máximo de su zona y el helper sigue activo, pero en realidad ya debería estar en otro mapa.

### Campos de cada zona

- **Nombre** — identificador de la zona. Debe contener palabras del nombre del mapa del juego (ej: "Kanturu", "Atlans") para que la verificación de mapa funcione correctamente
- **Nivel min / max** — rango de nivel en que aplica esta zona
- **Trigger level** — nivel exacto en que el bot warpea a esta zona por primera vez
- **Reintentos por muertes/helper desactivado** — cuántas veces el bot re-warpea a esta zona antes de rotar a la siguiente del mismo nivel. Por defecto 3

### Rotación de zonas del mismo nivel

Si tenés varias zonas con el mismo rango de nivel (por ejemplo dos spots distintos de Kanturu 2), el bot las rota cuando una agota sus reintentos:

- Zona A agota sus reintentos → rota a Zona B, resetea el contador de A
- Zona B agota sus reintentos → rota de vuelta a A, resetea todos los contadores
- El ciclo continúa indefinidamente hasta que el personaje sube de nivel y activa el trigger de la siguiente zona

### Checkboxes por zona

- **activo** — si está desmarcado, el bot ignora esa zona. Las zonas inactivas se muestran en gris oscuro
- **test** — si está desmarcado, se saltea esa zona al hacer Test Secuencia

### Botones de zona

- **cap** — captura la posición del mouse con cuenta regresiva de 3s. El botón **↩** deshace la última captura
- **+ paso** — agrega un paso a la walk sequence
- **⏺ Grabar** — graba la secuencia de pasos en el juego (ver sección Grabador)
- **⧉ Duplicar** — crea una copia exacta de la zona justo debajo
- **↑ ↓** — reordena zonas
- **Copiar a...** — copia esta zona a otros perfiles
- **X** — elimina la zona

### Botones de la pestaña

- **Guardar zonas** — guarda todos los cambios del perfil activo
- **+ Agregar zona** — agrega una zona vacía al final
- **Migrar todo a...** — copia toda la configuración del perfil actual a otros perfiles seleccionados
- **↑ Exportar** — guarda las zonas del perfil activo en un archivo JSON
- **↓ Importar** — carga zonas desde un archivo (pregunta si reemplazar o agregar al final)
- **★ Guardar como Default** — guarda las zonas actuales como perfil default con un nombre, para aplicar a personajes nuevos
- **✕ Borrar Default** — elimina un perfil default (siempre queda al menos uno)

---

## Grabador de secuencias

Permite grabar la walk sequence de una zona moviéndote en el juego en lugar de capturar coordenadas a mano.

### Cómo usar

1. Seleccioná la ventana del personaje en el combo VENTANA
2. En la zona que querés configurar, presioná **⏺ Grabar**
3. El bot ajusta la cámara automáticamente y muestra el popup de grabación
4. Presioná **F6** para iniciar la grabación — aparece el overlay rojo en el cursor
5. Posicioná el mouse en el primer punto del recorrido y presioná **F6** — el bot clickea ese punto en el juego y graba el paso
6. Esperá el tiempo que tarda el personaje en llegar, luego presioná **F6** en el siguiente punto
7. El tiempo entre cada F6 se graba automáticamente como los segundos de ese paso
8. Presioná **F7** para terminar la grabación
9. El bot pregunta si querés reemplazar la secuencia actual o agregar los pasos al final

> El grabador usa el mismo ajuste de cámara automático que el bot al iniciar, para que el personaje arranque desde la misma perspectiva.

---

## Perfiles por personaje

Cada personaje tiene su propio perfil guardado por nombre de ventana. Al seleccionar una ventana nueva por primera vez, se crea automáticamente un perfil copiando el default activo.

### Perfiles default

Los defaults son configuraciones base para aplicar a personajes nuevos. Aparecen en el combo de VENTANA como `[ [Default] Nombre ]` — podés seleccionarlos para ver y editar sus zonas, pero no se pueden ejecutar con el bot.

Para crear un nuevo default: configurá las zonas de cualquier perfil y usá **★ Guardar como Default**.

---

## MultiBot — múltiples cuentas

El MultiBot atiende cada ventana de forma secuencial: hace lo que corresponde en cada una y pasa a la siguiente indefinidamente.

### Cómo usarlo

1. Abrí todas las cuentas en el juego
2. Presioná **↺** para detectar todas las ventanas
3. Configurá cada cuenta seleccionándola en el combo VENTANA y ajustando sus zonas y región de level
4. Presioná **▶ Ejecutar Bot** → seleccioná las ventanas y presioná **▶ Ejecutar**

### Qué hace por cada ventana

1. Trae la ventana al frente y espera el foco
2. En la primera pasada ajusta la cámara automáticamente (solo una vez por sesión)
3. Lee el nivel del personaje
4. Evalúa las tres condiciones en orden: trigger de level → estado del helper → verificación de mapa
5. Actúa según corresponda y pasa a la siguiente ventana

---

## Ajuste de cámara automático

Al iniciar el bot o al abrir el grabador, el bot realiza una secuencia automática para posicionar la cámara:

1. Doble click con la rueda del mouse (centra la cámara)
2. Mantiene la rueda y arrastra hacia arriba (inclina la vista)
3. Hace zoom out con la rueda durante ~3 segundos

Esto se ejecuta **una sola vez por ventana** al iniciar — no se repite en los ciclos siguientes.