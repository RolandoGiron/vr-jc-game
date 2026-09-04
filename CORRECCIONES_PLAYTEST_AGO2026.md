# Correcciones del playtest — 25 ago 2026

Proyecto Unity: `C:\Users\rolan\test_mcp_unity`
Todos los cambios son de codigo (C#). No hace falta tocar la escena ni importar assets.
Tras compilar, basta con darle a Play.

---

## 1. "Al apretar E aparece un punto morado que solo se vuelve estrella al mover el mouse"

**Causa (dos):**
1. La "chispa" era una esfera con `Shader.Find("Standard")`. El proyecto es URP, donde
   ese shader no existe -> material rosa/morado de error.
2. La estrella real (`SM_Estrella`) no se instanciaba hasta el primer *drag*.

**Arreglo** — `Assets/_Base/Scripts/Story01/Beat1_Star.cs` (reescrito):
- Al pulsar **E** nace directamente **la estrella ya formada**, con material emisivo URP
  (`FXKit.Emissive`), luz puntual propia y estallido de chispas.
- Nace en la mano con un *pop*, **sube sola** al cielo sobre el establo (2,2 s) y solo
  entonces aparecen los tres Reyes Magos.
- Despues, clic izquierdo sostenido la guia por el firmamento (nunca baja de 4,5 m).
- Centelleo: giro lento + latido de la luz.
- Si `SM_Estrella.glb` faltara, se genera una estrella de 5 puntas con volumen por codigo.
- El visual se cuelga como hijo `PROP_Star_Visual`, que es el sufijo que `ArtDresser`
  reconoce como "ya vestido": antes el `PolishDirector` metia una **segunda** estrella
  dentro de la primera en cada rescaneo.

## 2. "Al tapar al burro la manta se ve doblada"

**Arreglo** — nuevo `Assets/_Base/Scripts/Polish/ClothDrape.cs` + `Beat2_Stable.cs`:
- Al soltar la manta en la zona del burro, se oculta el modelo doblado y se **genera una
  manta tendida** (malla procedural de 22x16) ajustada a las medidas reales del animal:
  plana sobre el lomo, cayendo por los dos flancos, con pliegues y borde ondulado.
- Es de doble cara (se ve tambien desde abajo) y lleva textura de franjas tejidas
  generada al vuelo (rojo con ribete crema), acabado mate tipo lana.

## 3. "No se encuentra la lampara ni se que hacer para colgarla"

**Arreglo** — `Beat2_Stable.cs`:
- La lampara esta **encendida**: material emisivo, luz puntual de 9 m y chispas.
- Se coloca sobre un **pedestal de piedra** a la entrada del establo (1.5, 0.62, 0.35),
  a la altura de la mano y visible desde la posicion inicial del jugador.
- Donde antes solo habia un `BoxCollider` invisible ahora hay **viga de madera, gancho de
  hierro y un aro dorado que late** marcando el punto exacto donde soltarla. El aro
  desaparece con un destello al colgarla.
- Se refresca la cache visual del `InteractableBase` tras vestir la lampara, para que el
  resaltado al apuntar funcione.

## 4. "Lumi tampoco me indica como hacerlo"

**Causa:** las pistas de Lumi eran `Debug.Log`. En pleno juego nadie ve la consola.

**Arreglo** — `Core/LumiGuide.cs` y `Core/StoryBeat.cs`:
- Nueva **pista en pantalla** (abajo, siempre visible) con el objetivo del beat activo;
  destella cuando Lumi la repite.
- Nuevo campo `StoryBeat.firstHintDelay`: los beats 1 y 2 dan la primera pista a los
  **8 s** en vez de a los 30.
- El beat 2 **reescribe la pista segun lo que falte** (heno 1/3, lampara, manta) y apunta
  a Lumi hacia ese objetivo concreto.

## 5. "En la entrada, a la par de las tablas hay libreros modernos"

**Causa:** `SetDressing.Hub()` sembraba 6 copias de `SM_Estanteria`, un librero moderno
lleno de libros encuadernados.

**Arreglo** — nuevo `Assets/_Base/Scripts/Hub/HubBackdrop.cs`:
- Fuera `SM_Estanteria`.
- **Estantes de madera tosca** (postes, travesanos y tablones) con **papiros enrollados**:
  cilindros de papiro con varillas de madera en los extremos y cintas rojas, apilados en
  dos alturas por balda.
- **Mesas de piedra** con un rollo abierto, tintero y calamo, y rollos de repuesto.
- **Anforas de barro** (`SM_Jarra`) entre los muebles.

## 6. "El fondo deberia ser mas impresionante"

**Arreglo** — `HubBackdrop.cs` + nuevo `Polish/NightSkyKit.cs`:
- **Hub:** nave con dos columnatas de 5 columnas unidas por **arcos de medio punto**,
  arquitrabe corrido, muros con pilastras, techo artesonado con vigas, alfombra
  ceremonial hasta las Tablas, estandartes y **cuatro braseros con llama y luz
  parpadeante**.
- **Gran arco** de 9 m en el muro del fondo abierto a un **cielo estrellado**, con terraza
  lejana, balaustrada, columnas en silueta y dos braseros: el fondo gana profundidad real.
- **Cielo procedural** (`NightSkyKit`): gradiente horizonte-cenit, bruma calida, via lactea
  con estructura de nube, ~2600 estrellas (las brillantes con cruz de difraccion) y luna
  con halo. Se aplica como skybox en el Hub.
- **Nacimiento:** la cupula `SKY_Dome` se pinta con ese mismo cielo estrellado y se anaden
  **dunas lejanas en silueta** para tener horizonte.
- Los ~900 bloques de la sala comparten materiales por color (cache en `HubBackdrop.M`)
  para que el SRP Batcher los agrupe.

---

## Archivos tocados

| Archivo | Estado |
|---|---|
| `Assets/_Base/Scripts/Story01/Beat1_Star.cs` | reescrito |
| `Assets/_Base/Scripts/Story01/Beat2_Stable.cs` | reescrito |
| `Assets/_Base/Scripts/Core/LumiGuide.cs` | pista en pantalla + `firstHintDelay` |
| `Assets/_Base/Scripts/Core/StoryBeat.cs` | nuevo campo `firstHintDelay` |
| `Assets/_Base/Scripts/Polish/SetDressing.cs` | Hub sin libreros; Nacimiento con cielo y dunas |
| `Assets/_Base/Scripts/Polish/ClothDrape.cs` | **nuevo** |
| `Assets/_Base/Scripts/Polish/NightSkyKit.cs` | **nuevo** |
| `Assets/_Base/Scripts/Hub/HubBackdrop.cs` | **nuevo** |

Copias de seguridad de los dos beats originales: `Beat1_Star.cs.bak` y `Beat2_Stable.cs.bak`
en la carpeta temporal de la sesion (no en el proyecto).

## Como probar

1. Abrir `Assets/_Base/Scenes/Hub.unity` -> Play. Se ve la sala con columnata, el gran arco
   con el cielo estrellado y los estantes de papiros. Apuntar a un libro + G para entrar.
2. `Assets/_Base/Scenes/Story01_Nacimiento.unity` -> Play.
   - **E**: la estrella aparece ya formada y sube al cielo.
   - Clic izquierdo sostenido: guiarla sobre el establo hasta que lleguen los 3 Reyes.
   - Beat 2: seguir la pista de abajo. Heno con **G**, lampara del pedestal al aro dorado,
     manta al lomo del burro (queda tendida).

---

# Segunda ronda — playtest 2

## 7. "La manta queda suspendida encima del burro"

**Causa:** la tela se asentaba en `bounds.max.y` del animal entero. En un cuadrupedo ese
maximo son **las orejas**, no el lomo: la manta flotaba a la altura de la cabeza.

**Arreglo** — `Polish/ClothDrape.cs` (nuevo `FitToBack`):
- Se muestrean los vertices reales de la malla y se corta el animal en 24 franjas a lo
  largo de su eje mayor.
- Se identifica el **barril del cuerpo** (el tramo contiguo de franjas mas anchas), lo que
  descarta cuello y cabeza automaticamente.
- La manta se asienta sobre la altura de ESE lomo, con el ancho, el largo y la
  **orientacion** del animal (funciona igual si el modelo mira a X o a Z).
- Si no se puede leer la malla, la manta doblada simplemente se apoya y se avisa por
  consola en vez de quedar flotando.

## 8. "Las letras del hint se ven feas, texto blanco encima"

**Arreglo** — `Core/LumiGuide.cs`:
- El aviso pasa a ser un **cartucho** con marco de brasa: textura 9-slice generada al vuelo
  (bronce oscuro -> ascua -> llama -> oro) sobre fondo negro calido semitransparente.
- El marco **late**: mas rapido y brillante cuando Lumi acaba de repetir la pista.
- Texto en **dorado** (`#FFD675`, oro claro al destellar) con **contorno negro en 8
  direcciones**: legible sobre cualquier fondo, sin el bloque blanco de antes.
- Tamano de letra proporcional a la altura de pantalla (14-26 px).

## 9. "Cuando sale Jesus deberia iluminarse todo por un momento"

**Arreglo** — `Story01/Beat4_Cradle.cs` (reescrito) + `Polish/ScenePolish.GoldenReveal`:

Secuencia cumbre de 5 s al completar el gesto de cuna:

| Tiempo | Que pasa |
|---|---|
| 0.00-0.30 s | **Fogonazo**: destello calido a pantalla completa (82 %), luz de gloria de 0 a 16, estallido de chispas y onda expansiva desde el pesebre |
| 0.00-2.50 s | El bebe se eleva 90 cm y **toda la escena** cruza de noche azul a dorado: luz ambiente, niebla, sol, bloom (0.9 -> 2.1) y exposicion (0.1 -> 0.85) |
| 2.50-5.00 s | La gloria se asienta en luz calida estable (5.2), con motas doradas subiendo |

`ScenePolish.GoldenReveal(t)` toca todos los canales a la vez —antes solo subia la luna y
se leia como "han subido el brillo"—. El cuenta atras del gesto ahora se ve en el hint.

## 10. "Deberia haber mas animalitos alrededor del pesebre"

**Arreglo** — `Polish/SetDressing.Nativity()`:
- **6 ovejas** y **5 corderitos** repartidos en anillo alrededor del pesebre (decorativos,
  con `CharacterLife.Animal`, sin collider ni nombres `NPC_*`, asi que no interfieren con
  los animales acariciables que crea el Beat 2).
- 4 colocados a mano mirando al pesebre y un **segundo camello** descansando junto al portal.
- **Fogata de pastores** con luz calida parpadeante y pavesas subiendo.

## 11. "La estrella mas pequena, parada vertical y con destellos sutiles"

**Arreglo** — `Story01/Beat1_Star.cs`:
- Tamano de **0.90 m a 0.62 m**, y normalizada por su dimension **mayor** (antes se
  dividia por la altura: si el GLB venia tumbado, el resultado era enorme).
- **Se pone en vertical**: si la malla llega plana (altura < 60 % de su anchura) se gira
  -90 grados en X automaticamente.
- Nuevo `StarPose`: la mantiene **de cara al jugador** con un balanceo de 5 grados. Se
  elimino el giro sobre el eje Y, que era justo lo que la hacia verse "de canto".
- **Destellos sutiles**: estela continua de motas doradas que la acompana por el cielo, mas
  un fogonazo breve cada 2.6-5.4 s (chispas + subida de luz).
- La estrella **sigue centelleando toda la historia**, no solo durante el Beat 1.

## Archivos tocados en esta ronda

| Archivo | Estado |
|---|---|
| `Assets/_Base/Scripts/Polish/ClothDrape.cs` | reescrito (`FitToBack`) |
| `Assets/_Base/Scripts/Core/LumiGuide.cs` | HUD dorado con marco de brasa |
| `Assets/_Base/Scripts/Story01/Beat4_Cradle.cs` | reescrito (momento cumbre) |
| `Assets/_Base/Scripts/Polish/ScenePolish.cs` | `GoldenReveal` + `Boost` |
| `Assets/_Base/Scripts/Polish/SetDressing.cs` | rebano, corderos, camello y fogata |
| `Assets/_Base/Scripts/Story01/Beat1_Star.cs` | estrella menor, vertical y con destellos |
| `Assets/_Base/Scripts/Story01/Beat2_Stable.cs` | usa `ClothDrape.FitToBack` |

---

# Tercera ronda — playtest 3 (captura de los Reyes Magos)

## 12. "Siguen viendose las letras feas"

No era el cartucho de abajo (ese quedo bien) sino las **etiquetas 3D de los cofres**:
"ORO / manten G y gira el raton en circulos" y compania, en blanco y enormes.

**Causas (tres):**
1. `characterSize = size * 0.1` con `fontSize 64`: letras de medio metro de alto y
   carteles de casi 3 m de ancho. Se salian de pantalla ("MIRRA manten G qu...").
2. El shader de fuente de Unity (`GUI/Text Shader`) dibuja con **ZTest Always**: el texto
   atravesaba el establo, la pared y los personajes.
3. La etiqueta se colgaba del cofre con `SetParent`, y los cofres tienen escala **no
   uniforme** (0.5 x 0.35 x 0.35): al girar el billboard, el texto salia **estirado**.

**Arreglo** — `Core/GrayboxFactory.Label` (reescrito) + nuevo componente `LabelSign`:
- Cartel compacto: **panel negro con marco de brasa** y letra **dorada**. El panel usa el
  MISMO shader que la fuente, asi que texto y fondo comparten profundidad y culling y el
  cartel se lee como una pieza unica, no como texto suelto flotando.
- Letra ~2.5 veces mas pequena y el panel **se ajusta solo** al texto que contenga.
- **Tamano aparente constante**: escala con la distancia (limitada entre 0.75x y 2.2x), asi
  ni se come la pantalla de cerca ni resulta ilegible de lejos.
- **Se oculta** a mas de 16 m o cuando el objeto queda a la espalda.
- Ya **no se cuelga del objeto**: lo sigue por posicion (`LabelSign.target`), de modo que
  ninguna escala rara lo deforma. `DestroyLabels` sigue funcionando igual (busca tambien
  los carteles que apuntan a ese objeto).
- Beneficia a las **tres historias**: baculo, corderito, nina, balde, cuerdas, pez grunon,
  pedestal, piezas de lampara, pluma, jarra... todas usan esta misma funcion.
- Textos del Beat 3 acortados: "ORO / G + gira en circulos", "INCIENSO / sopla con B, 2 s",
  "MIRRA / G quieto, 3 s", y el cartel baja de 0.80 m a 0.55 m sobre el cofre.

## 13. Bonus visto en la captura: los cofres eran cubos de colores

`Beat3_Gifts` crea los cofres como `PROP_CofreOro` / `PROP_CofreIncienso` /
`PROP_CofreMirra`, pero el registro de `ArtDresser` solo tenia `PROP_GiftGold` /
`PROP_GiftIncense` / `PROP_GiftMyrrh`, que no los usa nadie. Resultado: los tres cofres se
quedaban como el **cubo violeta** y el **cubo naranja** que se ven en la captura.

**Arreglo** — `Polish/ArtDresser.cs`: anadidos los tres nombres reales al registro, con sus
modelos `SM_CofreOro`, `SM_CofreIncienso` y `SM_CofreMirra` (0.38 m).
Ademas `OpenChest` ya no pinta solo el Renderer del host (que queda apagado al vestir el
cofre): ahora enciende **todo el subarbol** con material emisivo y lanza chispas.

---

# Cuarta ronda — playtest 4

## 14. "Hay ovejas enterradas en el suelo"

**Causa:** `ArtDresser.Dress` asienta el modelo usando los bounds del host. Cuando el host
**no tiene Renderer** —y todo lo que crea `SetDressing.Scatter/Place` son GameObjects
vacios— se usaba una caja ficticia de 0.5 m de lado, cuyo minimo cae **25 cm por debajo**
del punto del host. Con `ArtAlign.Ground`, cada oveja, cordero, anfora y fogata decorativa
se enterraba esos 25 cm.

**Arreglo** — `Polish/ArtDresser.cs`: si el host no tiene Renderer, la caja de referencia
pasa a ser de **tamano cero**, asi la base del modelo se apoya exactamente en el punto del
host. Arregla de golpe el rebano del Nacimiento, las anforas del Hub y la fogata.

## 15. "El efecto luminico esta bien, pero luego deberia quitarse y quedar normal"

**Arreglo** — `Story01/Beat4_Cradle.cs`: la gloria deja de ser una rampa que se queda
arriba y pasa a ser una curva completa de 6.8 s:

| Fase | Duracion | Que hace |
|---|---|---|
| Subida | 2.2 s | noche azul -> dorado pleno (fogonazo incluido) |
| Meseta | 2.0 s | gloria plena: todos celebran |
| Retirada | 2.6 s | dorado -> **la noche de siempre** |

Al terminar se llama `GoldenReveal(0)` y se devuelve la luna a su color y su intensidad
originales: ambiente, niebla, sol, bloom y exposicion quedan **exactamente como estaban**.
El bebe vuelve a bajar al pesebre y solo se queda un **halo tenue** sobre el (luz de 1.1 y
radio 9 m), para que el nino siga brillando sin que la escena quede lavada de oro.

## 16. "Jose y Maria mostrar alegria, y todos los demas, reyes magos y animalitos"

**Arreglo** — nuevo `Polish/JoyReaction.cs`:
- Los modelos de Meshy no tienen esqueleto, asi que no hay animaciones que lanzar. La
  alegria se construye con `CharacterLife`, que ya sabe hacer **saltos con squash and
  stretch**, cabeceos y respiracion.
- `JoyReaction.CelebrateAround(pesebre, 22 m, ~10 s)` recorre la escena y hace celebrar a
  **Maria, Jose, los tres Reyes Magos, el burro, el buey, las ovejas acariciables, el
  rebano decorativo, los corderitos y los camellos**.
- A quien le faltaba `CharacterLife` (los Reyes Magos y los animales del Beat 2 nunca lo
  tuvieron) se le anade en ese momento, con el arquetipo correcto.
- Cada uno sube su pulso vital (respira mas rapido y se mueve mas), encadena **saltitos con
  ritmo propio** —mas altos en los animales, mas contenidos en las personas—, asiente cada
  dos saltos y suelta chispas doradas sobre su cabeza en el primero.
- Durante la celebracion nadie mira a la camara: todos estan a lo suyo.
- Al acabar, cada personaje **recupera sus valores originales**: la escena queda igual que
  antes, sin residuos.
