# Personajes con esqueleto y animaciones — 29 ago 2026

Sustituyen a las mallas estaticas de Meshy. Viven en
`Assets/_Base/Art/Models/Rigged/SK_*.glb` y los importa `com.unity.cloud.gltfast`
con su skin y sus AnimationClips.

## Como se generaron

Blender **no hace falta abierto**: el pipeline corre headless con Blender 4.2 instalado
como modulo de Python (`pip install bpy==4.2.0`, Python 3.11).

Codigo en `tools/blender/` (`autorig.py` + `run.py`):

```
importar GLB -> quitar la placa de suelo -> soldar -> decimar -> suavizar
-> MEDIR la malla -> construir esqueleto -> peso automatico -> animar -> exportar GLB
```

```
python3 run.py -- <origen.glb> SK_Nombre <tipo> <triangulos> <salida.glb> <carpeta_previews>
```

Tipos de rig: `human`, `quad`, `curled`, `bird`, `swaddle`.

## Inventario

| Modelo | Tipo | Huesos | Triangulos | Animaciones |
|---|---|---|---|---|
| SK_Maria | human | 15 | 248.000 -> 14.000 | Idle, Joy, Bow, Walk |
| SK_Jose | human | 15 | 222.000 -> 14.000 | Idle, Joy, Bow, Walk |
| SK_ReyMago1 | human | 15 | 570.000 -> 18.000 | Idle, Joy, Bow, Walk |
| SK_ReyMago2 | human | 15 | 236.000 -> 14.000 | Idle, Joy, Bow, Walk |
| SK_ReyMago3 | human | 15 | 361.000 -> 14.000 | Idle, Joy, Bow, Walk |
| SK_NinaNPC | human | 15 | 254.000 -> 12.000 | Idle, Joy, Bow, Walk |
| SK_Soldado | human | 15 | 395.000 -> 14.000 | Idle, Joy, Bow, Walk |
| SK_Burro | quad | 21 | 292.000 -> 6.000 | Idle, Joy, Walk, Graze |
| SK_Buey | quad | 21 | 305.000 -> 6.000 | Idle, Joy, Walk, Graze |
| SK_Camello | quad | 21 | 232.000 -> 6.000 | Idle, Joy, Walk, Graze |
| SK_Oveja | quad | 21 | 280.000 -> 4.500 | Idle, Joy, Walk, Graze |
| SK_Corderito | curled | 8 | 262.000 -> 4.000 | Idle, Joy, Sleep |
| SK_BebeJesus | swaddle | 3 | 160.000 -> 4.000 | Idle, Joy, Sleep |
| SK_Lumi | bird | 7 | 122.000 -> 5.000 | Idle, Joy, Flap |

Peso automatico (bone heat) en los 14: **0 vertices sin peso**.

Triangulos totales de los 14 personajes: ~136.000, frente a ~3,9 millones antes.

## Lo que costo iteraciones

- **La malla de Meshy viene fragmentada** en miles de trozos sueltos: la placa de suelo no se
  puede quitar por islas. Se detecta midiendo el area horizontal ocupada en franjas del 1% de
  la altura (la placa cubre ~1400 celdas de una rejilla 48x48; encima quedan ~60, las pezunas).
- **Presupuesto de triangulos**: a 7.000 la cara de Maria se destruye; a 14.000 queda como el
  original; a 26.000 no mejora. Los cuadrupedos aguantan 6.000.
- **Los personajes son chibi**: la cabeza puede ser el 27% de la altura y el cuello estar al 72%,
  no al 87% de un humano real. El cuello se mide como el minimo de anchura entre 0.50 y 0.88 de
  la altura; los hombros, el maximo justo debajo. El hueso `Head` debe cubrir TODA la cabeza o la
  capucha se abre en abanico al girarla.
- **Las rotaciones de la cadena se acumulan**: una reverencia de 16+14+12+10+7 grados sumaba 59 y
  desprendia la cabeza. La version buena suma ~26.
- **El cuello hay que medirlo**: en el camello el centroide frontal caia sobre el pecho y dejaba
  un hueso de cuello de 4 cm. Ahora el cuello va del pecho a la union cuello-cabeza y la cabeza
  de ahi al morro.
- **Las patas se escalan por proporcion**: el camello tiene patas del 64% de su altura y el burro
  del 48%; con las mismas rotaciones al camello se le retorcian. Hay un factor `leg_gain`.
- El corderito esta **echado** y el bebe es un fardo: no admiten rig de cuadrupedo ni de humano.
  Lumi es una **paloma**, con rig de ave y aleteo real.

## Integracion en Unity (hecha)

### Como llega el modelo rigueado al juego

**No hubo que cambiar ni una llamada de los beats.** La carga se redirige sola:

- `AssetProbe.LoadModel("Assets/_Base/Art/Models/SM_Burro.glb")` busca primero
  `Assets/_Base/Art/Models/Rigged/SK_Burro.glb` y solo cae al plano si no existe.
- `ArtDresser.LoadModel("SM_Burro")` hace lo mismo.

Asi que un prop sin version rigueada (heno, cofres, lampara) sigue cargando su `SM_` de
siempre, y los 14 personajes reciben su esqueleto automaticamente.

### Importador

`Assets/_Base/Scripts/Editor/RiggedImportSetup.cs`:

- Un `AssetPostprocessor` fuerza `animationMethod = 1` (**Legacy**) en todo lo que entre por
  `Models/Rigged/`. glTFast importa por defecto en Mecanim (2), que exigiria crear un
  AnimatorController por personaje.
- Menu **VRJC > Rigged > Forzar animacion Legacy y reimportar** para reparar lo ya importado.
- Menu **VRJC > Rigged > Listar clips de cada personaje** para comprobar que los clips estan
  y son legacy.

### Reproductor

`Assets/_Base/Scripts/Polish/CharacterAnimator.cs`: envoltorio del sistema `Animation` legacy.

- `Play("Joy")`, `PlayFor("Joy", 4.6f)` (vuelve solo al reposo), `PlayOnce("Bow")`,
  `SetSpeed("Flap", x)`, `ToIdle()`.
- Los clips se resuelven de forma **tolerante**: si el importador los nombra
  `SK_Burro|Joy` en vez de `Joy`, los encuentra igual.
- `Bow` se marca como no ciclico; el resto en bucle.
- El reposo por defecto es `Idle`, o `Sleep` si no hay Idle (bebe), o `Flap` en el caso de Lumi:
  su reposo es volar.

### Quien anima que

| Sitio | Antes | Ahora |
|---|---|---|
| `ArtDresser.Dress` | siempre `CharacterLife` | `CharacterAnimator` si hay esqueleto; `CharacterLife` solo si no lo hay |
| `Bootstrap01` (Maria, Jose) | `CharacterLife` | idem |
| `AssetProbe.Place` (camello, decorados) | nada | engancha el animador |
| `Beat1` Reyes Magos | se deslizaban | **caminan** (`Walk`) y pasan a `Idle` al llegar al pesebre |
| `Beat2` animales | quietos | reposo animado; al **acariciarlos** responden con `Joy` |
| `Beat2` ovejas que te siguen | se deslizaban | caminan mientras te siguen |
| `Beat4` bebe | quieto | `Sleep`; despierta a `Joy` en el momento cumbre y vuelve a dormir |
| `JoyReaction` | saltitos simulados | clip `Joy` real; los saltitos quedan de respaldo |
| `LumiGuide` | aleteo por escala | clip `Flap`, con la velocidad atada al esfuerzo del vuelo |

`CharacterLife` **no se ha borrado**: sigue siendo el respaldo para cualquier modelo que aun
sea malla estatica, y nunca convive con el animador (darian doble movimiento).

## Que hacer al abrir Unity

1. Dejar que importe `Assets/_Base/Art/Models/Rigged/` (38 MB, tarda un poco).
2. Menu **VRJC > Rigged > Listar clips de cada personaje** y comprobar la consola: cada
   SK_*.glb debe listar sus clips sin el aviso `(NO legacy!)`.
3. Si alguno sale en Mecanim, **VRJC > Rigged > Forzar animacion Legacy y reimportar**.
4. Play en `Story01_Nacimiento`.

---

# Ronda 2 de integracion (29 ago 2026)

## El nino termina en brazos de Maria

**Antes:** el bebe subia con la gloria y al retirarse la luz **volvia a bajar al pesebre**.
El playtest lo describio como "se llena de luz y se esconde nuevamente en el heno": el
momento cumbre no cerraba en ningun sitio.

**Ahora**, en `Beat4_Cradle`:

- Al empezar la retirada de la gloria, **Maria abre los brazos** (clip `Hold`).
- El nino no cae en linea recta: describe un **arco** desde lo alto hasta el hueco entre
  las manos de Maria, girando hasta quedar acostado.
- Al llegar, se **emparenta al hueso del pecho** de Maria y pasa a `Sleep`. A partir de ahi
  se mece con ella, porque `Hold` lleva un balanceo lateral suave.
- La luz de gloria cuelga del nino, asi que baja con el e ilumina a Maria al final.

El punto de acunado se calcula con los huesos `Hand.L` y `Hand.R` reales, no con una
posicion fija: funciona aunque cambie la altura o la pose del modelo. Si no hubiera huesos
(modelo sin riguear), el nino se posa junto a ella y se avisa por consola.

### Clip nuevo: `Hold`

Anadido al rig humano y reexportados los 7 humanos. Brazos recogidos en cuna, cabeza
inclinada mirando al nino y mecido lateral de ~4 s en bucle. Los Reyes Magos, Jose, la Nina
y el Soldado tambien lo tienen (sirve para llevar cualquier cosa en brazos).

## Los pergaminos parecen pergaminos

**Antes:** `ScrollCollectible` creaba literalmente una **capsula dorada** por codigo.

**Ahora:**

- Usa el modelo **SM_Pergamino**; si faltara, construye un rollo de papiro procedural con
  varillas de madera y cinta roja.
- **Levita y gira**, con emision calida, **luz propia** de 5,5 m (de noche o en el vientre
  del pez, un pergamino apagado no se ve) y motas doradas.
- Lleva un **cartel** "PERGAMINO DE LUZ / G para recogerlo" con el estilo del resto.
- Al recogerlo **sube y se desvanece** con un estallido de chispas, en vez de desaparecer
  de golpe.
- Collider propio: el modelo importado llega sin colliders y el raycast necesita algo que
  golpear.

## Bug encontrado de paso: faltaban 3 de los 9 pergaminos

`VerseCatalog` define `S1_1`, `S1_2` y `S1_3` (Isaias 9:6, Miqueas 5:2, Lucas 2:11) y las
Tablas de la Ley del Hub reservan sus tres primeras lineas, pero **nadie los colocaba en la
escena**: solo existian los 6 de las Historias 2 y 3. El contador iba a 9 y era imposible
pasar de 6, asi que la decima linea de las Tablas nunca podia encenderse.

- `Bootstrap01` crea ahora los 3 que faltaban: junto a la fogata de los pastores, a la
  entrada del establo y detras, junto al camello.
- Los tres Bootstrap piden el texto a `VerseCatalog.Key("S1_1")` en vez de copiarlo a mano.
  El progreso se guarda por hash del texto, asi que una coma de diferencia hacia perder el
  pergamino: ahora es imposible que se desincronicen.
