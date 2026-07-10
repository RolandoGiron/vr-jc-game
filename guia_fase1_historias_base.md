# Guía de ejecución — FASE 1: Producción graybox de las 3 historias base + Pipeline de Assets
## Proyecto "El Gran Libro: Aventuras de la Biblia VR" · vía MCP (Claude ↔ unityMCP)

**Propósito:** Runbook paso a paso para que cualquier modelo Claude (Sonnet, Opus, Fable) construya, dentro del Unity Editor del usuario y vía **unityMCP**, la versión graybox jugable de las 3 historias gratuitas del GDD (Nacimiento, Mar Rojo, Jonás) más el hub mínimo, y documenta el **pipeline completo de creación de assets** (3D, texturas, música, narración, SFX) con las herramientas y conectores MCP recomendados.

**Prerrequisito:** haber completado la PoC descrita en `guia_poc_unity_mcp.md` (gesto "separar las aguas" validado ✅ 07-jul-2026). Esta guía **extiende** ese proyecto y **hereda íntegro el Protocolo obligatorio de su sección 3** (un script a la vez, `read_console` tras cada script, `save_scene` antes de `play`, nunca dejar play activo, rutas relativas a `Assets/`, configuración de componentes por código en Bootstrap).

**Alcance de esta fase (qué SÍ y qué NO):**
- ✅ Las 3 historias completas en **graybox** (primitivas, materiales flat por código, input simulado por teclado) con todos sus beats narrativos jugables de inicio a fin.
- ✅ Hub mínimo funcional (3 "libros" que cargan cada historia).
- ✅ Pipeline de assets definido y carpetas listas para recibir arte/audio final.
- ❌ NO incluye: Meta XR SDK, hand tracking real, arte final, optimización Quest (eso es Fase 2 del GDD). Toda mecánica usa el **patrón de contrato desacoplado** validado en la PoC: la lógica de juego lee una interfaz (`SimInput`) que hoy implementa el teclado y mañana implementará `OVRHand` sin tocar el gameplay.

---

# PARTE I — PIPELINE DE ASSETS: CON QUÉ Y CÓMO CREAR CADA TIPO

## 1.1 Mapa de herramientas por tipo de asset

| Tipo de asset | Herramienta/conector recomendado | Alternativa manual/gratis | Formato final en Unity |
|---|---|---|---|
| **Modelos 3D low-poly** | Conector MCP de generación 3D (buscar en el directorio de Claude: *Meshy*, *Tripo3D* — ambos generan modelos desde texto/imagen y tienen integración MCP) | **Kenney.nl** (CC0, estilo perfecto para el GDD), **Quaternius** (CC0), Synty POLYGON (Asset Store, pago), o Blender manual | `.fbx` / `.glb` → `Assets/_Base/Art/Models/` |
| **Texturas / concept art** | Conector MCP de generación de imágenes (buscar en directorio: generadores tipo *Ideogram*, *Recraft*, etc.) para concepts y texturas planas | El estilo toon del GDD (§5.1) casi **no necesita texturas**: colores flat + gradientes por shader. Atlas simples se hacen en Krita/GIMP (gratis) | `.png` → atlas 2048², ASTC 6x6 → `Assets/_Base/Art/Textures/` |
| **Música original** | **Suno** o **Udio** (generación por prompt; verificar licencia comercial del plan), **AIVA** (licencia comercial clara, ideal para publicar en la Store) | Compositor freelance; librerías con licencia (Artlist, Epidemic) | `.ogg` (Vorbis, streaming) → `Assets/_Base/Audio/Music/` |
| **Narración y voces (Lumi, narrador, versículos ES/EN/PT)** | **ElevenLabs** (tiene servidor MCP oficial: TTS multiidioma + efectos de sonido por prompt — el conector más útil de todo el pipeline de audio) | Grabación propia + Audacity (gratis) | `.ogg` streaming → `Assets/_Base/Audio/VO/{es,en,pt}/` |
| **Efectos de sonido (SFX)** | ElevenLabs SFX (por prompt: "whale passing overhead, deep rumble") | **Freesound.org** (filtrar CC0), **jsfxr/Bfxr** para SFX de UI, Audacity para editar | `.wav` (PCM/decompressed) → `Assets/_Base/Audio/SFX/` |
| **Diagramas de diseño** | **Mermaid Chart MCP** (ya conectado) — máquinas de estados de cada historia, flujo de beats | draw.io | Documentación, no entra al build |

**Recomendación práctica para ESTA fase:** graybox = primitivas de Unity + materiales flat por código (cero assets externos, cero bloqueos). Los assets finales entran en Fase 2 por el flujo de la sección 1.3. Para probar el pipeline temprano, importar 1 modelo de prueba (una oveja de Kenney o generada por conector 3D) y 1 clip de voz de Lumi (ElevenLabs) al final de esta fase.

## 1.2 Especificaciones que TODO asset debe cumplir (del GDD §5)

- Modelos: ≤ 10k tris personaje principal, 1 material por objeto, estilo low-poly chibi, escala real (1 unidad Unity = 1 m).
- Texturas: atlas 2048² por historia, compresión ASTC 6x6, sin normal maps pesados.
- Música: Vorbis calidad 0.4–0.6, `Load Type: Streaming`.
- SFX cortos: `Decompress On Load`; SFX largos/ambientes: `Compressed In Memory`.
- Audio espacial: mono para fuentes 3D (la espacialización HRTF la hace el Meta XR Audio SDK en Fase 2); estéreo solo para música.
- Naming: `SM_` mallas estáticas, `SK_` skinned, `T_` texturas, `MUS_`, `VO_`, `SFX_` audio. Todo bajo `Assets/_Base/`.

## 1.3 Flujo de importación de assets externos vía MCP (paso a paso)

1. `[USUARIO]` descarga o genera el asset (o Claude lo genera con el conector correspondiente si devuelve un archivo/URL de descarga) y lo coloca en una carpeta local, p. ej. `C:/GranLibro_Assets/entrantes/`.
2. `[USUARIO]` indica a Claude la ruta absoluta de esa carpeta y la ruta absoluta del proyecto Unity.
3. Claude usa **Filesystem MCP** (`read_file` para verificar, `write_file`/copiar) para mover el archivo a la subcarpeta correcta de `Assets/_Base/...` del proyecto. Alternativa: `unityMCP:import_asset` si el bridge lo soporta con rutas absolutas.
4. `[USUARIO]` da foco a la ventana de Unity (dispara el reimport automático).
5. Claude verifica con `unityMCP:get_asset_list` que el asset aparece, y `read_console` para descartar errores de importación.
6. La configuración de import (compresión, load type) que el bridge no exponga se anota en un checklist para que el `[USUARIO]` la aplique en el inspector (una sola vez por tipo, usando Presets de Unity).

## 1.4 Lista maestra de assets a producir para las 3 historias (backlog de arte/audio)

**Historia 1 — Nacimiento:** establo, pesebre, bebé, María/José (estáticos en graybox), 3 Reyes Magos, camello, burro, buey, 3 ovejas, estrella, 3 cofres (oro/incienso/mirra), lámpara, heno (x6), manta. Audio: MUS_belen (nana orquestal), VO narrador beats 1–4, SFX: chispa, coro cumbre, ronroneo buey, balido.
**Historia 2 — Mar Rojo:** báculo, paredes de agua (malla animada en F2), lecho marino con corales, 6 peces, tortuga, ballena (solo sombra+audio), corderito, niña NPC, 6 soldados silueta cómica, carro. Audio: MUS_exodo (épica), VO beats, SFX: rugido de agua proporcional, splash gigante, ballena overhead.
**Historia 3 — Jonás:** barco cubierta, balde, cuerdas, gran pez (interior estómago-caverna), 8 medusas, pez gruñón compañero, mástil roto, barriles (x5), piezas de lámpara (x3), úvula gigante, pluma, playa + planta. Audio: MUS_jonas (juguetona), VO beats, SFX: tormenta, trago, estornudo cómico, ambiente bioluminiscente.
**Transversal:** Lumi (paloma-luz), pergaminos de luz (x9), fogata, hub biblioteca (estanterías, Gran Libro, 3 libros-diorama), UI diegética mínima.

---

# PARTE II — RUNBOOK DE CONSTRUCCIÓN GRAYBOX (ejecutable por cualquier modelo)

## 2. Protocolo (recordatorio de cumplimiento OBLIGATORIO)

Aplicar íntegra la sección 3 de `guia_poc_unity_mcp.md`. Adiciones específicas de esta fase:

- **A.** Cada historia vive en su propia escena: `Assets/_Base/Scenes/Hub.unity`, `Story01_Nacimiento.unity`, `Story02_MarRojo.unity`, `Story03_Jonas.unity`. Nunca mezclar historias en una escena.
- **B.** Todo el input pasa por `SimInput.cs` (contrato único). **Prohibido** leer `Input.GetKey` desde scripts de gameplay: solo `SimInput` lo hace. Así la migración a VR toca 1 archivo.
- **C.** Cada escena tiene su `BootstrapXX.cs` (materiales, cámara, componentes) y un `StoryDirector` que encadena los beats. Un beat no arranca hasta que el anterior dispara `OnBeatComplete`.
- **D.** Commit de git del `[USUARIO]` al cerrar cada FASE (checkpoint reversible).
- **E.** Claude no ve la pantalla: tras cada `play`, el `[USUARIO]` describe en una frase lo que ve; los scripts deben reportar TODOS los hitos por `Debug.Log` para que `read_console` sirva de telemetría (patrón validado en la PoC).

## 3. Contrato de input simulado (compartido por las 3 historias)

`Assets/_Base/Scripts/Core/SimInput.cs` — singleton. Mapeo teclado → gesto VR futuro:

| Propiedad del contrato | Teclado (graybox) | VR (Fase 2) | Usado en |
|---|---|---|---|
| `float handSeparation` (0–1) | mantener **ESPACIO** (sube gradual) | distancia entre palmas | Mar Rojo beat 2 |
| `bool slam` | **X** | brazos abajo rápido | Mar Rojo beat 4 |
| `bool palmOpen` | **E** | palma abierta detectada | Nacimiento beat 1 |
| `Vector3 pointerPos` + `bool dragging` | mouse + clic izq. | pinch + raycast de dedo | arrastrar estrella, tocar peces |
| `bool grabPressed` | clic der. / **G** | grip / pinch | balde, heno, cofres, pluma, bebé |
| `bool blow` | **B** | pico de volumen del micrófono | incienso, shofar futuro |
| `float rubIntensity` | machacar **F** | fricción palma-palma | encender lámpara (Jonás) |
| `bool crouch` | **C** | altura del visor < umbral | esquivar mareas (Jonás) |
| `float gentleness` (0–1) | velocidad del mouse invertida | suavidad del movimiento de mano | acariciar animales |

## 4. FASE A — Núcleo compartido y Hub

### A.0 Auditoría (idéntica a Fase 0 de la PoC)
`get_scene_info` → `get_hierarchy` → `list_scripts (Assets)` → `read_console(show_errors)`. Documentar inventario. Criterio: consola sin errores.

### A.1 Scripts core (crear UNO a la vez, verificando consola tras cada uno)

| # | Script (`Assets/_Base/Scripts/Core/`) | Responsabilidad |
|---|---|---|
| 1 | `SimInput.cs` | Contrato de la sección 3. Singleton `DontDestroyOnLoad`. Log al activar cada gesto. |
| 2 | `StoryDirector.cs` | Lista ordenada de `StoryBeat` (clase base abstracta con `Activate()`, `IsComplete`, evento `OnBeatComplete`). Avanza secuencialmente, log "BEAT n/4 COMPLETADO". Al terminar el último: log "HISTORIA COMPLETADA" + retorno al Hub tras 5 s. |
| 3 | `LumiGuide.cs` | Esfera amarilla emisiva que orbita al jugador. Timer: si el beat activo lleva >30 s sin progreso → log pista contextual (string definido por cada beat) + Lumi se mueve hacia el objetivo. Celebración al completar beat (órbita rápida + log). |
| 4 | `ScrollCollectible.cs` | Pergamino (cápsula dorada rotante). Al tocarlo con `pointerPos`+`grabPressed`: log del versículo + se registra en `GameProgress`. 3 por historia. |
| 5 | `GameProgress.cs` | Singleton persistente: historias completadas, pergaminos (9 total). `PlayerPrefs` para persistir. |
| 6 | `InteractableBase.cs` | Base común: detección de pointer sobre el objeto (raycast desde cámara), estados hover/grab, virtual `OnGrab/OnRelease/OnPoke`. |
| 7 | `SnapZone.cs` | Trigger que acepta un objeto por tag; al soltar dentro: imán suave a la posición, log "SNAP: {objeto} → {zona}", evento consumible por los beats. |

### A.2 Escena Hub
1. `new_scene` → `Assets/_Base/Scenes/Hub.unity`.
2. `create_object`: `P_MainCamera` (0, 1.6, -3), `ENV_Sun`, `ENV_Floor` (plano 3×3, "nube"), 3 cubos-libro: `HUB_Book01_Nacimiento` (-1.2, 1, 0), `HUB_Book02_MarRojo` (0, 1, 0), `HUB_Book03_Jonas` (1.2, 1, 0), `GameManager` (EMPTY).
3. Script `HubManager.cs` (`Scripts/Hub/`): materiales por código (azul noche / turquesa / verde), apuntar+`grabPressed` sobre un libro → log "ABRIENDO {historia}" → `SceneManager.LoadScene`. Libros de historias completadas: material dorado (lee `GameProgress`).
4. `attach_script` de `SimInput`, `GameProgress`, `HubManager` al `GameManager`. **Nota:** las escenas de historia deben estar en Build Settings para que `LoadScene` funcione — si el bridge no lo expone, `[USUARIO]` las añade una vez (File → Build Settings → Add Open Scenes) o el código usa `LoadScene` por índice tras confirmarlo.
5. Ciclo: `save_scene → play → read_console → stop`. Criterio: clic en cada libro loggea la apertura (aunque la escena destino aún no exista, el intento debe loggearse controladamente con try/catch).

**Checkpoint git del `[USUARIO]`.**

## 5. FASE B — Historia 1: El Nacimiento de Jesús (graybox, 4 beats)

### B.1 Escena
`new_scene` → `Story01_Nacimiento.unity`. Objetos: `P_MainCamera` (0,1.6,-4), `ENV_Moon` (luz azul tenue, intensidad 0.4), `ENV_Ground` (plano), `ENV_Stable` (3 cubos formando establo en 0,0,3), `NPC_Maria`, `NPC_Jose` (cápsulas estáticas), `PROP_Pesebre` (cubo hueco), `SKY_Dome` (esfera invertida escala 40 — material azul noche por código), `GameManager`. Guardar.

### B.2 Beats (un script por beat, en `Scripts/Story01/`, verificando consola tras cada uno)

| Beat | Script | Mecánica graybox | Criterio de completado |
|---|---|---|---|
| 1 | `Beat1_Star.cs` | `palmOpen` (E) hace aparecer `FX_Spark` (esfera emisiva pequeña) en frente de cámara; `grabPressed`+soltar la lanza al cielo → se convierte en `PROP_Star` (esfera dorada grande en el domo). Luego `dragging` sobre el domo mueve la estrella; 3 cápsulas `NPC_ReyMago` siguen la trayectoria proyectada al suelo (agente simple: mover hacia la proyección). | Los 3 reyes llegan al trigger del establo. Log hitos: chispa creada / estrella encendida / reyes 1-2-3 llegaron. |
| 2 | `Beat2_Stable.cs` | 6 cubos `PROP_Hay` arrastrables (grab con mouse) a `SnapZone` del pesebre (necesita 3), `PROP_Lamp` a snap-zone alta, `PROP_Blanket` al `NPC_Burro`. Animales (`NPC_Buey`, `NPC_Oveja` x3): pasar el pointer despacio (`gentleness > 0.6`) = caricia → log "el buey ronronea"; las ovejas persiguen la cámara 5 s. | 3 snaps completados (heno×3 cuenta como 1, lámpara, manta). |
| 3 | `Beat3_Gifts.cs` | 3 cofres: **Oro** = mantener `grabPressed` y girar mouse en círculo (acumulador angular ≥ 720°); **Incienso** = `blow` (B) sostenido 2 s; **Mirra** = grab + mantener el pointer dentro de una franja estrecha 3 s (verter con precisión). Cada cofre abierto → log del dato educativo (§4 GDD) y snap ante el pesebre. | 3 regalos colocados. |
| 4 | `Beat4_Cradle.cs` | Mantener ESPACIO 3 s (gesto cuna bimanual) frente al pesebre → `PROP_Baby` (cápsula pequeña) se eleva suave; crossfade de iluminación por código (subir intensidad + cambiar color a dorado — simula el cambio de lightmaps del GDD §5.2); log "MOMENTO CUMBRE". | Gesto sostenido completo. |

### B.3 Ensamblaje y prueba
`Bootstrap01.cs` en `GameManager`: materiales (paleta azul noche + dorados), componentes, registra los 4 beats en `StoryDirector`, coloca 3 `ScrollCollectible` (detrás del establo, junto al camello, en el techo). Ciclo completo de la Fase 5 de la PoC hasta: consola muestra la secuencia BEAT 1→2→3→4 + "HISTORIA COMPLETADA" sin errores, y el `[USUARIO]` confirma visualmente cada mecánica. **Checkpoint git.**

## 6. FASE C — Historia 2: Moisés y el Mar Rojo (reutiliza la PoC, 4 beats)

### C.1 Migración de la PoC
1. Abrir `Assets/Scenes/SampleScene1.unity` (donde el bridge guardó la PoC), `save_scene` hacia `Assets/_Base/Scenes/Story02_MarRojo.unity` (o recrear la escena si el bridge no permite "guardar como": los 3 scripts de la PoC ya existen y se re-adjuntan).
2. Refactor mínimo: `SimulatedHands.cs` pasa a leer de `SimInput` (mismo contrato `separation/gestureActive/slamThisFrame`) para unificar. `read_console` tras el cambio.

### C.2 Beats nuevos (`Scripts/Story02/`)

| Beat | Script | Mecánica graybox | Criterio |
|---|---|---|---|
| 1 | `Beat1_Staff.cs` | `grabPressed` (G) cerca de `PROP_Staff` (cilindro) lo "invoca" a la mano (sigue a la cámara). Al acercarse al mar: log "el báculo vibra" con frecuencia creciente (simula háptica). | Báculo en mano + jugador en zona de orilla. |
| 2 | *(existente)* `SeaPartingController` | Ya validado en la PoC. Integrarlo como `StoryBeat` (wrapper `Beat2_Parting.cs` que escucha su hito 100%). | Apertura 100%. |
| 3 | `Beat3_Crossing.cs` | Corredor entre paredes: mover cámara con WASD (locomoción graybox). `FishFollower.cs`: 3 esferas-pez siguen `pointerPos` si está cerca. Tocar pared con pointer → log "ondas". `NPC_Lamb` (grab para cargar, soltar en el otro extremo) y `NPC_Girl` (se acerca si mantienes pointer sobre ella 1 s "dar la mano", luego te sigue). Sombra de ballena: cubo aplanado que cruza sobre el corredor + log "ballena sobre ti" (marcador del futuro audio espacial). | Jugador + corderito + niña cruzan el trigger final. |
| 4 | `Beat4_Closing.cs` | Spawn de 6 cápsulas `NPC_Soldier` que avanzan por el corredor (velocidad cómica, tropiezos = rotaciones random). `slam` (X) → cierre rápido de aguas (ya soportado por la PoC) + `FX_Splash` (esferas blancas balísticas) + log "ARCOÍRIS". Los soldados quedan detrás del muro cerrado (desactivarlos). | Slam con todos los aliados a salvo. |

### C.3 Ensamblaje
`Bootstrap02.cs`: paleta turquesa/coral, corredor de 20 m, registra beats 1–4, 3 pergaminos (bajo un coral, tras la pared L, en la orilla final). Ciclo de prueba completo. Criterio de cierre: secuencia 4/4 en consola sin errores + confirmación visual del `[USUARIO]`. **Checkpoint git.**

## 7. FASE D — Historia 3: Jonás y la Gran Ballena (4 beats)

### D.1 Escenas
Dos "burbujas" (GDD §5.1) en una misma escena, separadas 100 m: `ZONE_Deck` (cubierta) y `ZONE_Belly` (estómago). La transición "tragado" = fade por código (esfera negra que envuelve la cámara 1.5 s) + teletransporte del rig. `new_scene` → `Story03_Jonas.unity`.

### D.2 Beats (`Scripts/Story03/`)

| Beat | Script | Mecánica graybox | Criterio |
|---|---|---|---|
| 1 | `Beat1_Storm.cs` | La cubierta (plano) se balancea (sin física, seno animado — confort). `PROP_Bucket` grab + llevarlo a zona de agua + volcarlo fuera de borda ×5 (contador). 2 `PROP_Rope`: mantener grab 2 s c/u. Decisión: acercarse al borde + `grabPressed` = saltar → transición tragado (fade suave, log "confort: sin aceleraciones"). | 5 baldes + 2 cuerdas + salto. |
| 2 | `Beat2_Belly.cs` | Estómago-caverna (cubos verdes emisivos). 5 `PROP_Barrel` con Rigidbody de gravedad reducida (flotante). Cada 20 s: evento "marea" — log de aviso 2 s antes, los barriles reciben impulso en cámara lenta (`Time.timeScale` 0.5 durante 2 s), el jugador debe `crouch` (C) o recibe empujón cómico (log). 8 `PROP_Jelly` (esferas): tocar con pointer → se iluminan 5 s. `NPC_GrumpyFish`: tocarlo 3 veces → te sigue (compañero). | Sobrevivir 2 mareas + reclutar al pez. |
| 3 | `Beat3_Lamp.cs` | 3 `PROP_LampPiece` flotando en puntos altos/bajos: grab y llevar a `SnapZone` del pedestal. Con las 3: `rubIntensity` (machacar F) llena barra 0–1 → lámpara encendida (emisivo) → grab y snap en gancho alto → 3 `PROP_Verse` (quads emisivos) aparecen en las paredes, tocarlos loggea el versículo. | Lámpara colgada + 3 versículos revelados. |
| 4 | `Beat4_Sneeze.cs` | `PROP_Feather` grab → llevarla a `PROP_Uvula` (cápsula rosa colgante) y "cosquillas" = mover el pointer sobre ella 3 s acumulados → cuenta regresiva cómica en log → lanzamiento balístico suavizado del rig (curva precalculada, 2 s, fade parcial — confort) hasta `ZONE_Beach` (tercer área). Epílogo: `PROP_Plant` + grab de `PROP_WaterJug` sobre ella → la planta escala ×3. Log "HISTORIA COMPLETADA". | Expulsión + planta regada. |

### D.3 Ensamblaje
`Bootstrap03.cs`: paleta verde bioluminiscente, gravedad flotante de la zona belly (configurar Rigidbodies por código: `drag` alto, gravedad escalada), registra beats, 3 pergaminos (bajo un barril, dentro de una medusa, en la playa). Ciclo de prueba completo + confirmación visual. **Checkpoint git.**

## 8. FASE E — Integración transversal y validación final

1. **Retorno al hub:** verificar que completar cada historia vuelve a `Hub.unity` y el libro correspondiente se muestra dorado (persistencia `GameProgress`).
2. **Muro de versículos mínimo:** en el hub, `HUB_VerseWall` (quad) muestra por log/escala cuántos pergaminos van (n/9).
3. **Prueba de pipeline de assets (opcional pero recomendada):** importar 1 modelo (sección 1.3) y sustituir una primitiva (p. ej. la oveja); importar 1 clip VO de Lumi y reproducirlo con `AudioSource` creado por Bootstrap. Documentar cualquier fricción del flujo.
4. **Checklist de criterios de éxito** (Claude redacta, `[USUARIO]` confirma uno a uno):
   - [ ] Hub carga las 3 historias y refleja progreso.
   - [ ] Nacimiento: 4/4 beats, estrella arrastrable, 3 regalos con interacciones distintas.
   - [ ] Mar Rojo: apertura proporcional intacta tras el refactor + cruce con aliados + cierre con slam.
   - [ ] Jonás: tormenta→tragado→sandbox→lámpara→expulsión→planta, con transiciones de confort.
   - [ ] 9 pergaminos coleccionables y persistentes.
   - [ ] Lumi da pista a los 30 s en todos los beats.
   - [ ] 0 errores en consola en un playthrough completo de cada historia.
5. `save_scene` final ×4 escenas, `git commit -m "Fase 1 graybox completa"` (`[USUARIO]`).
6. **Documentación:** generar con Mermaid Chart MCP el diagrama de flujo de beats de cada historia y el diagrama de clases del core (`SimInput`/`StoryDirector`/`StoryBeat`/`InteractableBase`).

## 9. Manejo de errores (adicional a la tabla de la guía PoC)

| Síntoma | Causa probable | Solución |
|---|---|---|
| `LoadScene` falla ("scene not in build settings") | Escenas no registradas | `[USUARIO]` añade las 4 escenas en Build Settings una vez |
| El beat no avanza | Evento `OnBeatComplete` no disparado | Buscar en consola el último hito loggeado; el hito faltante señala la condición rota |
| Objetos flotantes atraviesan paredes (Jonás) | Colliders sin configurar | Añadir/ajustar colliders en `Bootstrap03.Awake()` por código |
| Time.timeScale queda en 0.5 tras una marea | Excepción durante el evento | try/finally restaurando timeScale; verificar con log "timeScale restaurado" |
| El asset importado se ve rosa | Shader incompatible | Asignar material por código en Bootstrap con `Shader.Find("Universal Render Pipeline/Lit")` o Unlit |

## 10. Prompt de arranque para otro modelo (copiar y pegar)

> Ejecuta la guía `guia_fase1_historias_base.md` adjunta a este proyecto. Prerrequisito: la PoC de `guia_poc_unity_mcp.md` ya está hecha. Cumple estrictamente el Protocolo de la sección 3 de la guía PoC más las adiciones A–E de esta guía: todo input vía `SimInput`, un script a la vez con `read_console` tras cada uno, `save_scene` antes de `play`, nunca dejar play activo, y reportar hitos por `Debug.Log`. Comienza por la FASE A.0 (auditoría) y repórtame el inventario antes de avanzar. Ejecuta las fases en orden A→B→C→D→E, pidiéndome confirmación visual al final de cada ciclo de prueba y un commit de git al cerrar cada fase. Yo seré tus ojos en play mode.

---

*Fin de la guía Fase 1 · v1.0 · Julio 2026 · Extiende: guia_poc_unity_mcp.md · GDD de referencia: GDD.md*
