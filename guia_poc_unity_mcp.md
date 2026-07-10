# Guía de ejecución: Prueba de Concepto en Unity vía MCP (Claude)

**Propósito:** Runbook paso a paso para que cualquier modelo Claude (Sonnet, Opus, Fable) pueda construir la PoC del GDD dentro del Unity Editor del usuario usando el servidor **unityMCP**, sin intervención manual salvo donde se indica explícitamente `[USUARIO]`.

**Estado verificado al momento de crear esta guía:** conexión Claude ↔ unityMCP ↔ Unity Editor activa y funcional (`get_scene_info` respondió correctamente).

---

## 1. Arquitectura de herramientas y cómo se interconectan

```
┌─────────────┐   MCP (stdio/sse)   ┌──────────────┐   Editor API   ┌──────────────┐
│   Claude    │ ◄─────────────────► │  unityMCP    │ ◄────────────► │ Unity Editor │
│ (cualquier  │                     │  (servidor   │                │ (proyecto    │
│  modelo)    │                     │   bridge)    │                │  abierto)    │
└─────────────┘                     └──────────────┘                └──────────────┘
      │
      ├──► Filesystem MCP ──► archivos locales del usuario (assets externos, docs, texturas)
      ├──► Mermaid Chart MCP ──► diagramas de arquitectura/flujo del GDD (documentación)
      └──► Google Drive MCP ──► respaldo del GDD y documentación de diseño
```

**Roles de cada herramienta:**

| Herramienta | Rol en el pipeline | Cuándo usarla |
|---|---|---|
| `unityMCP` | Motor principal: escenas, objetos, scripts, prefabs, play mode, consola | Todo el trabajo dentro de Unity |
| `Filesystem MCP` | Leer/copiar assets externos (sprites, modelos, audio) hacia la carpeta `Assets/` del proyecto | Fase de importación de assets |
| `Mermaid Chart MCP` | Generar diagramas de máquinas de estado, flujo de gameplay y arquitectura de clases | Documentación y planeación |
| `git` (terminal del usuario) | Control de versiones y puntos de restauración | Antes de cada fase |

**Regla de interconexión clave:** unityMCP solo ve rutas **relativas a `Assets/`**. Si un asset externo debe entrar al proyecto, se copia con Filesystem MCP a la carpeta `Assets/` del proyecto y Unity lo importa automáticamente al recuperar el foco.

---

## 2. Prerrequisitos — comandos que ejecuta el `[USUARIO]` en su máquina

Estos pasos son manuales porque requieren acceso a tu sistema operativo:

### 2.1 Unity Editor
1. Tener **Unity Hub** con un editor **2021.3 LTS o superior** (recomendado 2022.3 LTS).
2. El **proyecto debe estar abierto** en el editor durante toda la sesión. Si Unity está cerrado, todas las llamadas MCP fallan.
3. No dejar el editor en **Play Mode** mientras Claude crea o edita scripts (la recompilación en play causa comportamiento inestable).

### 2.2 Servidor unityMCP
Ya está instalado y verificado. Si en algún momento las herramientas dejan de responder:
```bash
# Verifica que el proceso del bridge esté corriendo (según tu instalación):
# Si lo instalaste vía uv/python:
uv run server.py
# Si es el paquete de Unity: Window > MCP Server > Start (dentro del editor)
```
Luego reconecta el conector en Claude (Configuración → Conectores → unityMCP → Reconectar).

### 2.3 Control de versiones (fuertemente recomendado)
Desde la raíz del proyecto Unity:
```bash
git init
curl -o .gitignore https://raw.githubusercontent.com/github/gitignore/main/Unity.gitignore
git add .
git commit -m "Estado inicial antes de PoC via MCP"
```
Y **antes de cada fase** de esta guía:
```bash
git add . && git commit -m "Checkpoint: fin de fase N"
```
Esto permite revertir cualquier cambio que haga Claude con `git checkout .`

### 2.4 Assets externos (solo si el GDD los requiere)
Si la PoC usa sprites/modelos/audio descargados, colócalos en una carpeta accesible y dile a Claude la ruta; Claude los copiará con Filesystem MCP a `Assets/_PoC/Art/` (o los generará como primitivas si no hay assets — recomendado para PoC).

---

## 3. Protocolo obligatorio para el modelo ejecutor (Claude)

Cualquier modelo que ejecute esta guía DEBE seguir estas reglas. Son el resultado de las limitaciones reales del bridge:

1. **Verificar antes de asumir.** Al inicio de cada sesión: `get_scene_info` → `get_hierarchy` → `list_scripts` → `read_console`. Nunca asumir el estado del proyecto entre sesiones.
2. **Después de CADA `create_script` o `update_script`:** esperar la recompilación y llamar `read_console` con `show_errors: true`. Si hay errores CS####, corregirlos con `update_script` antes de continuar. **Nunca encadenar múltiples scripts sin verificar compilación.**
3. **El nombre de la clase debe ser idéntico al nombre del archivo** (`PlayerController.cs` → `public class PlayerController`). Unity no compila si difieren.
4. **Guardar antes de probar:** `save_scene` → `play`. Nunca entrar a play con la escena sin guardar.
5. **Nunca dejar el play mode activo** al terminar un turno: siempre `stop` después de leer la consola.
6. **Rutas siempre relativas a `Assets/`** en todas las herramientas (`Assets/Scenes/PoC.unity`, `Assets/_PoC/Scripts/...`).
7. **Nombres únicos de GameObjects.** `attach_script` y `create_prefab` buscan por nombre; objetos duplicados causan ambigüedad. Usar prefijos (`P_Player`, `ENV_Ground`).
8. **Un ciclo de iteración = ** editar script → `read_console` (compilación OK) → `save_scene` → `play` → `read_console` (runtime OK) → `stop` → siguiente cambio.
9. **Si una herramienta falla con timeout,** el editor probablemente está compilando o sin foco. Reintentar UNA vez tras informar al usuario; si persiste, pedir al usuario que haga clic en la ventana de Unity.
10. **Configuración de componentes vía código, no vía editor.** El bridge no expone edición granular de componentes (Rigidbody, Colliders, etc.), así que TODA configuración se hace desde los scripts: un script `PoCBootstrap.cs` con `[RuntimeInitializeOnLoadMethod]` o métodos `Awake()` que agregan y configuran componentes con `AddComponent<T>()`. Este es el patrón central de toda la PoC.

---

## 4. Fases de ejecución

### FASE 0 — Auditoría del entorno (Claude, automático)
| # | Acción | Herramienta | Parámetros |
|---|---|---|---|
| 0.1 | Estado de la escena actual | `unityMCP:get_scene_info` | — |
| 0.2 | Jerarquía completa | `unityMCP:get_hierarchy` | — |
| 0.3 | Scripts existentes | `unityMCP:list_scripts` | `folder_path: "Assets"` |
| 0.4 | Consola limpia | `unityMCP:read_console` | `show_errors: true` |

**Criterio de salida:** sin errores en consola; inventario documentado en el chat.

### FASE 1 — Estructura del proyecto (Claude, automático)
No existe herramienta de "crear carpeta"; las carpetas se crean implícitamente:

| # | Acción | Cómo |
|---|---|---|
| 1.1 | Crear `Assets/_PoC/Scripts/` | `create_script` con `script_folder: "_PoC/Scripts"` creando el primer script real |
| 1.2 | Crear `Assets/_PoC/Scenes/` | `new_scene` con `scene_path: "Assets/_PoC/Scenes/PoC_Main.unity"` |
| 1.3 | Crear `Assets/_PoC/Prefabs/` | Se crea al guardar el primer prefab con `create_prefab` |

**Convención de nombres:** todo bajo `_PoC/` para poder borrar la prueba de concepto sin tocar el resto del proyecto.

### FASE 2 — Escena base (Claude, automático)
| # | Acción | Herramienta | Parámetros ejemplo |
|---|---|---|---|
| 2.1 | Nueva escena | `new_scene` | `scene_path: "Assets/_PoC/Scenes/PoC_Main.unity"` |
| 2.2 | Cámara | `create_object` | `type: "CAMERA", name: "P_MainCamera", location: [0, 8, -10], rotation: [35, 0, 0]` |
| 2.3 | Luz | `create_object` | `type: "LIGHT", name: "ENV_Sun", rotation: [50, -30, 0]` |
| 2.4 | Suelo | `create_object` | `type: "PLANE", name: "ENV_Ground", scale: [5, 1, 5]` |
| 2.5 | Guardar | `save_scene` | — |

### FASE 3 — Scripts núcleo (Claude, automático; **depende del GDD**)
Patrón por cada sistema del GDD (movimiento, spawner, puntuación, UI, etc.):

1. `create_script` con `script_folder: "_PoC/Scripts"`, `script_name: "<Sistema>"`, `content:` código C# **completo** (nunca fragmentos).
2. `read_console` con `show_errors: true` → corregir con `update_script` si hay errores.
3. Repetir para el siguiente sistema. **Uno a la vez.**

Script obligatorio independientemente del GDD — `PoCBootstrap.cs`: configura al inicio del juego todos los componentes que el bridge no puede editar (Rigidbodies, colliders, materiales físicos, capas, UI Canvas creado por código, etc.). Ejemplo de esqueleto:

```csharp
using UnityEngine;

public class PoCBootstrap : MonoBehaviour
{
    void Awake()
    {
        // Configura físicas, cámara, UI y referencias entre sistemas por código.
        // Ejemplo: var rb = GameObject.Find("P_Player").AddComponent<Rigidbody>();
        // rb.constraints = RigidbodyConstraints.FreezeRotation;
    }
}
```

### FASE 4 — Ensamblaje de la escena (Claude, automático)
| # | Acción | Herramienta |
|---|---|---|
| 4.1 | Crear GameObjects de gameplay (jugador, enemigos, pickups) como primitivas | `create_object` |
| 4.2 | Adjuntar scripts | `attach_script` (`object_name`, `script_name`) |
| 4.3 | Crear objeto `GameManager` vacío y adjuntarle `PoCBootstrap` | `create_object` tipo `EMPTY` + `attach_script` |
| 4.4 | Convertir entidades repetibles en prefabs | `create_prefab` (`prefab_path: "Assets/_PoC/Prefabs/X.prefab"`) |
| 4.5 | Poblar la escena | `instantiate_prefab` con posiciones |
| 4.6 | Guardar | `save_scene` |

### FASE 5 — Ciclo de prueba e iteración (Claude, automático)
Repetir hasta que la PoC cumpla los criterios del GDD:

```
save_scene → play → (esperar 5-10 s de juego) → read_console → stop
    │
    ├─ errores/NullReference → update_script → read_console → repetir
    └─ sin errores → verificar con el usuario que el comportamiento visible es correcto
```

**Importante:** Claude no ve la pantalla del juego. El `[USUARIO]` es los ojos: tras cada `play`, describe en una frase qué viste ("el cubo se mueve pero atraviesa el suelo"). Claude traduce eso a correcciones de código.

### FASE 6 — Validación y cierre
| # | Acción | Quién |
|---|---|---|
| 6.1 | Checklist de criterios de éxito del GDD (uno por mecánica core) | Claude redacta, usuario confirma |
| 6.2 | `save_scene` final + limpiar consola | Claude |
| 6.3 | `git add . && git commit -m "PoC funcional"` | `[USUARIO]` |
| 6.4 | (Opcional) Build standalone: cargar `unityMCP:build` vía tool_search y ejecutar para la plataforma objetivo | Claude, con confirmación del usuario |
| 6.5 | Diagrama final de arquitectura con Mermaid Chart MCP para documentar la PoC | Claude |

---

## 5. Manejo de errores comunes

| Síntoma | Causa probable | Solución |
|---|---|---|
| Timeout en cualquier herramienta | Unity compilando o minimizado | `[USUARIO]` da foco a la ventana de Unity; Claude reintenta |
| `error CS0116 / CS8025` tras create_script | Contenido malformado o clase ≠ nombre de archivo | `update_script` con el archivo completo corregido |
| `attach_script` no encuentra el objeto | Nombre duplicado o typo | `get_hierarchy` para verificar el nombre exacto |
| El objeto no se comporta como se espera en play | Componente sin configurar | Configurarlo en `PoCBootstrap.Awake()` por código |
| `NullReferenceException` en runtime | Referencia entre scripts no asignada | Reemplazar campos serializados por `GameObject.Find()` o singleton en `Awake()` |
| Cambios de script no surten efecto | Editor no recompiló | `[USUARIO]` da foco a Unity (Ctrl+R fuerza refresh); Claude vuelve a `read_console` |

---

## 6. Tareas específicas del GDD "El Gran Libro" — ✅ EJECUTADO (07-jul-2026)

**PoC elegida (definida por el propio GDD, §6 Fase 0 / Riesgo #1):** prototipo gris del gesto bimanual "separar las aguas" del Mar Rojo (§2.3.2) con manos simuladas, lógica desacoplada del hardware VR.

### 6.1 Scripts implementados (en `Assets/_PoC/Scripts/`)

| Script | Objeto | Responsabilidad |
|---|---|---|
| `SimulatedHands.cs` | `P_Rig` | Simula las 2 manos (esferas cian/magenta). Expone el contrato `separation` / `gestureActive` / `slamThisFrame`. Incluye AutoDemo (validación sin input) y modo manual: **ESPACIO** = separar manos, **X** = brazos abajo. En producción se sustituye por Meta Interaction SDK implementando el mismo contrato. |
| `SeaPartingController.cs` | `GameManager` | Núcleo: mapea la separación de manos a apertura **proporcional y suavizada** de las 2 paredes de agua (requisito del GDD: control total = sensación de poder). El mar se mantiene abierto al soltar el gesto; el slam lo cierra con fuerza. Reporta hitos 25/50/75/100% a consola. |
| `PoCBootstrap.cs` | `GameManager` | Configura por código lo que MCP no puede tocar en el inspector: materiales (agua azul, arena) y encuadre de cámara. |

### 6.2 Escena graybox

`WaterWall_L` y `WaterWall_R` (cubos 1×10×20 m, el muro de agua "de 20 m" del GDD), `ENV_Seabed` (lecho marino), `P_Rig` (jugador a 1.4 m de altura), `P_MainCamera`, `ENV_Sun`, `GameManager`. Nota: el bridge guardó la escena activa como `Assets/Scenes/SampleScene1.unity` en lugar de la ruta `_PoC/Scenes/` — renombrar/mover manualmente en el editor si se desea orden.

### 6.3 Resultado de la validación (logs reales de play mode)

Ciclo completo verificado por consola, **0 errores**: bootstrap OK → gesto iniciado → apertura proporcional 25% → 50% → 75% → 100% en ~4 s (tiempo objetivo del GDD) → sostener → slam → "MAR CERRADO. Ciclo completo de la mecánica insignia EJECUTADO CON ÉXITO."

### 6.4 Criterios de éxito de la PoC

1. ✅ La apertura es proporcional y suavizada respecto a la separación de las manos.
2. ✅ El gesto sostenido de ~4 s abre el corredor completo.
3. ✅ El mar se mantiene abierto al soltar el gesto (diseño del GDD).
4. ✅ El slam ("brazos abajo") cierra las aguas con velocidad diferenciada.
5. ⏳ Sensación en VR real — requiere fase siguiente (Meta XR SDK + visor).

### 6.5 Próximos pasos (orden recomendado)

1. **[USUARIO]** Probar manualmente en el editor: Play → mantener ESPACIO despacio/rápido → X. Confirmar que la sensación visual es correcta.
2. Sustituir `SimulatedHands` por un provider `XRHands`/`OVRHand` con el mismo contrato (requiere Meta XR All-in-One SDK instalado y visor por Link — paso manual del usuario).
3. Paredes de agua: reemplazar cubos por malla animada con vertex shader (GDD §5.3) + audio de rugido proporcional a la velocidad de apertura.
4. Añadir "Lumi" (pista si el jugador no completa el gesto en 30 s, GDD §2.1).
5. Prefabs de rezagados (corderito/niña) para la micro-interacción de cruce (GDD §2.3.3).

---

## 7. Prompt de arranque para otro modelo (copiar y pegar)

> Ejecuta la guía `guia_poc_unity_mcp.md` adjunta. Sigue estrictamente el Protocolo de la sección 3, especialmente: verificar consola tras cada script, un script a la vez, guardar antes de play, y nunca dejar play activo. Comienza con la Fase 0 (auditoría) y repórtame el estado antes de avanzar. El GDD con las tareas específicas está en la sección 6 / adjunto. Yo seré tus ojos durante las pruebas en play mode.
