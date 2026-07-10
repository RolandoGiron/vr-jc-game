# GDD — Documento de Diseño Conceptual y Plan de Proyecto
## Juego VR educativo de historias bíblicas para Meta Quest

**Versión:** 0.1 (Concepto) · **Fecha:** Julio 2026 · **Plataforma objetivo:** Meta Quest 2 / 3 / 3S / Pro
**Motor sugerido:** Unity (URP) + Meta XR All-in-One SDK

---

# 1. VISIÓN GENERAL Y CONCEPTO CORE

## 1.1 Elevator pitch

*"No leas la Biblia. Vívela."* Un juego VR episódico donde cada historia bíblica es un mundo diorama vivo, colorido y táctil que **solo avanza si el jugador actúa**: separa el mar con sus propios brazos, guía la Estrella de Belén con la mano, y escapa del vientre del gran pez resolviendo puzzles. Diversión primero, aprendizaje integrado, cero espectador pasivo.

## 1.2 Títulos tentativos (3 opciones)

| Opción | Título | Por qué funciona |
|---|---|---|
| A | **El Gran Libro: Aventuras de la Biblia VR** | El "Gran Libro" es además el hub del juego (la metáfora central). Escalable como marca de franquicia ("El Gran Libro: Navidad", etc.). |
| B | **Manos de Milagro (Miracle Hands)** | Comunica la propuesta única: los milagros ocurren *con tus manos* (hand tracking). Gancho perfecto para el tráiler de la Store. |
| C | **BibleLand VR** | Nombre corto, memorable, internacional, con vibra de "parque temático" que refuerza el tono colorido tipo Pixar. |

**Recomendación:** Opción A para mercado hispano/familiar, con B como subtítulo de marketing ("El Gran Libro — donde tus manos hacen milagros").

## 1.3 Público objetivo

- **Primario:** Niños de 6 a 12 años y familias que juegan juntas (modo "co-piloto": un adulto puede ver la partida vía casting y participar con preguntas).
- **Secundario:** Escuelas dominicales, iglesias y colegios cristianos — sesiones de 10–15 min por historia, ideales para clase. (Oportunidad B2B: licencias educativas, ver §3.5).
- **Terciario:** Público general curioso por narrativa interactiva y experiencias VR "wholesome" (género en crecimiento en Quest Store).
- **Rating objetivo:** IARC 3+/E. Sin violencia gráfica (incluso David vs. Goliat se resuelve con humor y física de caricatura).

## 1.4 Estilo visual y artístico

**Dirección de arte: "Vitral en movimiento"** — Low-poly estilizado y pulido, con la calidez de Pixar y la legibilidad de *Moss* o *Vacation Simulator* (referencias probadas de rendimiento nativo en Quest):

- **Paleta:** Colores saturados y luminosos; cada historia tiene su paleta firma (Belén = azules nocturnos + dorados cálidos; Mar Rojo = turquesas y corales; Jonás = verdes bioluminiscentes).
- **Personajes:** Proporciones "chibi-amable": cabezas grandes, ojos expresivos, siluetas leíbles a distancia. Animación exagerada tipo cartoon (squash & stretch).
- **Materiales:** Shaders flat/toon con gradientes suaves y rim-light falso (barato en GPU); nada de PBR pesado. Los "momentos milagro" usan partículas doradas y luz volumétrica *fake* (billboards) como firma visual.
- **Escala:** Mundos diorama a escala real para el jugador — todo está al alcance de la mano; ese es el lenguaje de diseño: *si lo ves, puedes tocarlo*.

---

# 2. MECÁNICAS DE JUEGO EN VR (GAMEPLAY INTERACTIVO)

## 2.1 Filosofía de interacción

**Regla de oro: la historia no avanza sola.** Cada beat narrativo está bloqueado detrás de una acción física del jugador. Soporte dual completo:

- **Controles Touch:** agarre físico (grip), lanzamiento con física real, háptica rica (el báculo "vibra" al acercarse al mar).
- **Hand Tracking 2.x:** gestos naturales como verbos de juego — empujar, separar, sostener, saludar, aplaudir. Los milagros *siempre* tienen versión de gesto con manos porque es más memorable ("yo separé el mar con mis manos").
- **Confort:** teletransporte + movimiento suave opcional, viñeta dinámica, alturas jugable sentado o de pie. Sesiones de 10–15 min por capítulo (atención infantil + rotación en aulas).
- **Narrador guía:** una lucecita-paloma ("Lumi") acompaña, da pistas si el jugador se atasca >30 s y celebra los logros. Nunca hay pantalla de "game over": fallar produce reacciones cómicas y reintento inmediato.

## 2.2 Historia gratuita 1 — El Nacimiento de Jesús

*Tono: acogedor, nocturno, mágico. Duración: ~12 min.*

1. **Enciende la Estrella:** el jugador recibe una chispa dorada en la palma (hand tracking detecta palma abierta), la sopla/lanza al cielo y luego **arrastra la Estrella de Belén con el dedo** por el firmamento para guiar a los Reyes Magos — literalmente dibuja la ruta sobre un cielo interactivo.
2. **Prepara el establo:** minijuego de orden acogedor — apilar heno en el pesebre, colgar una lámpara, abrigar al burro. Los animales reaccionan a caricias (el buey ronronea con háptica, las ovejas te siguen si les rascas la cabeza).
3. **Los regalos de los Reyes:** puzzle físico — cada cofre (oro, incienso, mirra) se abre con una interacción distinta: girar un mecanismo, encender el incienso soplando al micrófono, y verter la mirra con precisión. Al colocarlos ante el pesebre, cada regalo dispara un dato educativo (§4).
4. **Momento cumbre:** el jugador levanta con ambas manos (gesto de cuna) al bebé para presentarlo — la escena entera se ilumina, coro espacial 3D envolvente. Es el "screenshot moment" del capítulo.

## 2.3 Historia gratuita 2 — Moisés y el Mar Rojo

*Tono: épico, aventura. Duración: ~15 min. Es el DEMO estrella para tráilers.*

1. **El báculo vivo:** arma-herramienta del capítulo. Con Touch: grip + háptica creciente cerca del agua. Con hand tracking: se invoca cerrando el puño.
2. **LA mecánica insignia — Separar las aguas:** de cara a un muro de agua de 20 m, el jugador junta ambas manos al frente y las **separa lentamente hacia los lados** (gesto bimanual sostenido, ~4 segundos). El mar se abre en tiempo real proporcional al gesto — si abre los brazos despacio, el agua ruge y se aparta despacio. Control total = máxima sensación de poder. (Fallback Touch: mantener ambos gatillos y separar los mandos.)
3. **Caminar por el lecho marino:** pasillo entre dos paredes de agua vivas con peces, tortugas y una ballena que pasa como sombra (audio espacial arriba y a los lados — el momento más inmersivo del juego). Micro-interacciones en ruta: tocar la pared de agua hace ondas, hay peces curiosos que te siguen el dedo, y debes **ayudar a cruzar a rezagados** (cargar un corderito, dar la mano a una niña — el hand tracking brilla aquí).
4. **Cierre con tensión:** el ejército aparece detrás (siluetas cómicas, no amenazantes para el rating); el jugador debe llegar al otro lado y **bajar los brazos con fuerza** para cerrar las aguas — splash gigante con partículas y arcoíris.

## 2.4 Historia gratuita 3 — Jonás y la Gran Ballena

*Tono: humor + asombro. Duración: ~12 min. El capítulo más "juguetón".*

1. **Tormenta en cubierta:** el jugador achica agua con un balde físico, sujeta cuerdas y toma la decisión narrativa (saltar al mar) tirándose *él mismo* por la borda — transición de tragado en primera persona (diseñada con cuidado de confort: oscurecimiento suave, sin aceleraciones violentas).
2. **Dentro del gran pez — nivel sandbox:** un estómago-caverna bioluminiscente con física flotante: barriles, un mástil roto, medusas que iluminan al tocarlas, un pez gruñón atrapado que se vuelve tu compañero. El jugador **esquiva "mareas" internas** (el pez se sacude — objetos vuelan en cámara lenta) agachándose físicamente.
3. **Puzzle de la luz:** para "hablar con Dios" hay que reconstruir una lámpara: pescar piezas flotantes, encenderla frotando las manos (gesto de fricción con hand tracking) y colgarla en lo alto. La luz revela versículos brillando en las "paredes" del estómago (coleccionables §4).
4. **¡Expulsión!:** secuencia física cómica — el jugador provoca el estornudo del gran pez haciéndole **cosquillas a la úvula gigante** con una pluma, y sale disparado a la playa (movimiento balístico suavizado + splash). Remate: replantar y regar la planta de Jonás en la playa como epílogo tranquilo.

---

# 3. ESTRUCTURA DE CONTENIDO Y MODELO DE MONETIZACIÓN (FREEMIUM)

## 3.1 Distribución en Meta Horizon Store

- **App base GRATIS** con las 3 historias completas (no demos: completas). Esto maximiza descargas, reviews y posicionamiento orgánico — el embudo clásico free-to-download que mejor funciona en Quest para contenido familiar.
- Compras dentro de la app vía **Meta IAP (Add-ons)**: historias individuales, packs y pase completo.
- Actualizaciones de contenido estacionales (Navidad y Pascua) para picos de descargas y featuring en la Store.

## 3.2 El Hub: "La Gran Biblioteca"

El menú principal es un espacio jugable, no un menú 2D:

- Una **biblioteca-museo flotante entre nubes**, cálida y luminosa, con un **Gran Libro central** en un atril. Cada historia es un **libro-diorama viviente**: al tomarlo de la estantería y abrirlo, un diorama miniatura de la historia emerge de sus páginas (previsualización jugable en miniatura). Meter la cabeza/manos en el diorama inicia la historia con una transición de "encogimiento" mágica.
- Las historias no compradas aparecen como **libros cerrados con broche dorado** que muestran su diorama-teaser de 15 segundos — la mejor tienda es dejar que el niño *vea* lo que hay dentro. Botón de compra integrado diegéticamente (el broche se abre al comprar).
- El hub crece con el progreso: trofeos interactivos de capítulos completados (una mini-ballena nadando en el aire, una estrella de Belén que puedes volver a encender), mascotas rescatadas que viven ahí, y el **Muro de Versículos** coleccionados (§4).
- Zonas del hub por "estantería": Antiguo Testamento, Nuevo Testamento, Parábolas, y una estantería vacía visible rotulada "Próximamente" (anticipación de DLC).

## 3.3 Contenido DLC — 8+ historias adicionales

Precio sugerido por historia: **USD 2.99–3.99**. Packs de 3 por **USD 7.99**. "Pase del Gran Libro" (todo + futuras): **USD 24.99**.

| # | Historia | Gran momento interactivo en VR |
|---|---|---|
| 1 | **El Arca de Noé** | Construir el arca martillando tablones a escala real y luego, bajo la lluvia, **guiar parejas de animales con las manos** (cada especie responde a un gesto distinto: silbar, llamar, cargar a los perezosos). Gestión caótica y adorable estilo Overcooked. |
| 2 | **David y Goliat** | **Honda con física real**: girar el brazo físicamente para acumular impulso y soltar en el momento justo. Goliat es enorme visto desde la escala de David (el jugador *siente* los 3 metros). Derrota de caricatura: cae como árbol talado y hace temblar el suelo (háptica). |
| 3 | **Las Murallas de Jericó** | El jugador marca el ritmo de la marcha con percusión corporal y **toca el shofar real** (soplar al micrófono + sostenerlo en alto). A la séptima vuelta, las murallas se desmoronan en cascada de física low-poly alrededor del jugador — 360° de espectáculo. |
| 4 | **Daniel en el foso de los leones** | Sigilo y ternura: **calmar leones acercando la mano lentamente** (el hand tracking mide la suavidad del movimiento — movimientos bruscos los alertan). Terminar durmiendo apoyado en la melena de un león gigante que respira (audio 3D + háptica de ronroneo). |
| 5 | **La Creación (Génesis)** | El jugador es el pincel: **pintar la luz separándola de la oscuridad con un gesto**, esculpir montañas levantando las manos, sembrar bosques lanzando semillas, y soplar vida a los animales. Sandbox creativo por "días" — el capítulo más artístico y rejugable. |
| 6 | **José y la túnica de colores** | **Teñir y tejer la túnica en un telar interactivo** (elección de colores libre — personalización que luego se luce en el hub), y después descifrar los sueños del Faraón en un teatro de sombras manipulable: mover vacas y espigas de la escena onírica para interpretar el sueño. |
| 7 | **Jesús calma la tormenta** | El jugador rema y achica agua en una barca zarandeada por olas de verdad (física de oleaje + audio de trueno espacial)... hasta que aprende el gesto: **extender la palma y decir "calma"** — la tormenta se congela en el aire y se disuelve en estrellas. Contraste sensorial brutal: del caos al silencio total. Después: ¡caminar sobre el agua! |
| 8 | **La multiplicación de los panes y peces** | El jugador parte panes y peces que **se duplican en sus manos** (loop táctil hipnótico) y debe repartirlos lanzándolos con puntería suave a canastas que las familias levantan — servir a 5,000 personas se convierte en un minijuego frenético y generoso. |

**Backlog futuro (roadmap año 2):** La Torre de Babel (construcción cooperativa que se "desincroniza"), Sansón (fuerza física exagerada), El Buen Samaritano (parábola de decisiones), Pentecostés, Ester, El Hijo Pródigo.

## 3.4 Economía suave (sin paywalls oscuros)

- **Sin monedas virtuales ni loot boxes** — crítico para rating infantil, confianza de padres y políticas de la Store.
- Los coleccionables (§4) se ganan solo jugando y desbloquean cosméticos para Lumi y el hub (skins de la paloma, decoraciones de biblioteca). Refuerzan retención, no gasto.
- **Prueba gratuita por DLC:** los primeros 3 minutos de cualquier historia de pago son jugables desde su diorama-teaser ("modo ojeada").

## 3.5 Canal B2B educativo

Licencia "Aula" (precio por lote de dispositivos) con: modo profesor (tableta companion para lanzar historias en varios visores), guías de lección descargables por capítulo y reportes simples de progreso. Distribución vía Meta Quest for Business / ManageXR. Es un segundo motor de ingresos con LTV alto y churn bajo.

---

# 4. ENFOQUE EDUCATIVO Y GAMIFICACIÓN

**Principio: aprender es el premio, nunca la tarea.** El contenido educativo se descubre jugando, jamás interrumpe con texto obligatorio.

- **Pergaminos de Luz (coleccionables):** versículos clave escondidos como pergaminos brillantes en cada escena (3 por capítulo). Al atraparlos, el versículo se *escucha* narrado con música — y se cuelga en el Muro de Versículos del hub, donde puede volver a tocarse para releerse/escucharse.
- **Reliquias curiosas:** objetos históricos interactivos (una lámpara de aceite, un shofar, una red de pesca de la época) con datos de 15 segundos narrados por Lumi al manipularlos: cómo vivía la gente, qué comía, cómo eran los barcos de Jonás. Historia cultural, no solo relato.
- **Preguntas de fogata:** al final de cada capítulo, epílogo opcional junto a una fogata donde Lumi hace 3 preguntas de comprensión con respuestas físicas (lanzar la piedrita a la respuesta correcta). Acertar da cosméticos; fallar da una repetición amable de la escena clave, nunca castigo.
- **Logros con humor:** "Pastor Certificado" (acariciar a todas las ovejas), "Sushi No Gracias" (pasar 5 min dentro del pez), "Braceador Profesional" (separar el mar en menos de 5 s), "DJ de Jericó" (ritmo perfecto 7 vueltas). Los logros divertidos generan conversación y rejugabilidad.
- **Modo Familia/Aula:** subtítulos grandes multiidioma (lanzamiento: ES/EN/PT — el mercado cristiano de Brasil y LATAM es enorme y está desatendido en VR), narración regrabable por región, y "modo discusión" que pausa la historia en momentos clave para conversación guiada por el adulto.
- **Balance diversión/aprendizaje (regla 80/20):** 80% del tiempo el niño juega con física, mascotas y milagros; 20% es contenido educativo *opcional pero irresistible* porque brilla, suena y se colecciona.

---

# 5. REQUISITOS TÉCNICOS Y OPTIMIZACIÓN PARA META QUEST

**Objetivo de rendimiento:** 72 fps sólidos en Quest 2 (mínimo común), 90 fps en Quest 3/3S. Presupuesto de frame: ~13.8 ms (72 Hz).

## 5.1 Render y geometría

- **Unity URP + Vulkan**, Single Pass Instanced (ojo doble en una pasada). GPU es el cuello de botella típico en Quest: presupuesto de **200–350k triángulos en escena** (el estilo low-poly lo hace natural) y **50–120 draw calls** vía atlas de texturas por historia + static/GPU batching.
- **Shaders toon baratos:** un único ubershader flat con rim-light falso; prohibidos los shaders con múltiples luces en tiempo real por objeto. Nada de post-procesado costoso (bloom completo, SSAO); el "glow" de los milagros se hace con partículas aditivas y sprites.
- **Fixed Foveated Rendering (FFR)** nivel medio/alto en secuencias espectaculares (apertura del mar), y **Application SpaceWarp (ASW)** como red de seguridad en escenas de física intensa.
- **LODs agresivos + culling por zonas:** los dioramas son espacios pequeños por diseño — cada capítulo se divide en 2–3 "burbujas" con streaming aditivo de escenas (sin pantallas de carga: transiciones "libro que pasa página").

## 5.2 Iluminación

- **100% luz horneada (baked lightmaps) + Light Probes** para dinámicos. Una sola luz direccional en tiempo real como máximo, y solo cuando un milagro lo exige.
- Los cambios dramáticos de iluminación (mar abriéndose, estrella encendiéndose) se logran con **crossfade entre sets de lightmaps precalculados** y emisivos animados — impacto visual de cine a costo de móvil.

## 5.3 Física e interacción

- Física solo en objetos interactuables (pooling agresivo, dormir rigidbodies fuera de alcance). Las "paredes de agua" del Mar Rojo son mallas animadas por vertex shader (barato), no simulación de fluidos.
- **Meta Interaction SDK** para agarres, pokes y gestos — resuelve la paridad Touch/hand tracking sin duplicar código.

## 5.4 Audio 3D / espacial (pilar narrativo, no adorno)

- **Meta XR Audio SDK** con espacialización HRTF: la ballena que pasa *sobre* el jugador, el ejército que se acerca *por detrás*, el coro que envuelve el pesebre — el audio dirige la mirada del niño sin flechas en pantalla (regla de diseño: *el sonido es el señalador*).
- Reverb por zonas (estómago del pez = reverb húmedo y cercano; desierto = seco y abierto). Narración y música en streaming comprimido (Vorbis), SFX descomprimidos en memoria.
- Mezcla dinámica: la música baja cuando Lumi habla (ducking automático).

## 5.5 Presupuestos y pipeline

| Recurso | Presupuesto |
|---|---|
| Triángulos por escena | 200–350k |
| Draw calls | < 120 |
| Texturas | Atlas 2048², ASTC 6x6 |
| Memoria app | < 1.5 GB en runtime |
| Tamaño descarga base | < 2 GB (DLC ~300–500 MB c/u) |
| Personaje principal | ≤ 10k tris, 1 material |

- Perfilado continuo con **OVR Metrics + RenderDoc Meta Fork**; gate de build: ningún capítulo se aprueba bajo 72 fps en Quest 2.

---

# 6. PLAN DE PROYECTO (RESUMEN EJECUTIVO)

| Fase | Duración | Entregable |
|---|---|---|
| **0. Preproducción** | 6 semanas | Prototipo gris del gesto "separar el mar" (validar la mecánica insignia ANTES que todo), biblia de arte, vertical slice plan |
| **1. Vertical Slice** | 10 semanas | Mar Rojo completo con arte final + hub básico → material de pitch para Meta featuring |
| **2. Producción base** | 16 semanas | 3 historias gratuitas + hub completo + IAP + localización ES/EN/PT |
| **3. Beta y QA** | 6 semanas | Playtests con familias e iglesias (objetivo: 30 niños), pase de confort/rating, certificación Horizon Store |
| **4. Lanzamiento** | — | App base gratis + 2 DLC día uno (Noé y David: los de mayor reconocimiento de marca) |
| **5. Live ops** | continuo | 1 DLC cada 6–8 semanas, eventos de Navidad/Pascua, canal B2B |

**Equipo mínimo viable:** 1 game designer/director, 2 devs Unity XR, 2 artistas 3D low-poly, 1 sound designer (parcial), 1 narrador/guionista con asesor teológico (parcial). ~8–9 meses hasta lanzamiento.

**Riesgos clave:** (1) El gesto bimanual del mar debe sentirse perfecto — prototipar primero; (2) sensibilidad del contenido religioso — asesoría interdenominacional y tono universal centrado en las historias; (3) descubribilidad en la Store — la app gratuita + tráiler del Mar Rojo son la apuesta de adquisición.

---

*"El mejor cumplido posible: un niño que se quita el visor y dice — ¡yo separé el mar!"*
