# GUÍA DETALLADA — Fases 2, 3, 4 en Unity 6.4

---

## FASE 2: Reemplazar NPC_Maria

### Paso 1: Abrir escena y localizar el objeto
1. En **Project** (panel izquierda), navega a `Assets/_Base/Scenes/`
2. Doble-clic en `Story01_Nacimiento.unity` → se abre la escena
3. En **Hierarchy** (panel izquierda), busca `NPC_Maria` (debe estar en la lista)
4. Clic sobre `NPC_Maria` → se selecciona (queda subrayado en azul)

### Paso 2: Ver el modelo actual en Inspector
1. Con `NPC_Maria` seleccionado, ve al panel **Inspector** (derecha)
2. Verás componentes como:
   - Transform (posición X, Y, Z)
   - MeshFilter (el mesh actual, probablemente una cápsula graybox)
   - MeshRenderer (material aplicado)
3. **Nota:** Probablemente sea una primitiva (cápsula o cubo)

### Paso 3: Reemplazar el mesh por SM_Maria.glb
**Opción A (Más fácil — drag & drop):**
1. En **Project**, navega a `Assets/_Base/Art/Models/`
2. Encuentra `SM_Maria` (es un archivo .glb)
3. En el **Inspector** de `NPC_Maria`, busca el componente **MeshFilter**
4. Ve al campo **Mesh** dentro de MeshFilter
5. Arrastra `SM_Maria` desde Project al campo Mesh
   - O: clic en el círculo pequeño junto a "Mesh" → busca SM_Maria en la ventana que aparece

**Opción B (Reemplazar objeto completo):**
1. Selecciona `NPC_Maria` en Hierarchy
2. En Inspector, busca el componente **MeshFilter**
3. Clic en el botón con 3 puntos (⋮) junto a Mesh
4. Busca "SM_Maria" → selecciona → OK

### Paso 4: Verificar posición, escala, rotación
1. Con `NPC_Maria` aún seleccionado, mira el componente **Transform** en Inspector
2. Verifica:
   - **Position:** X=? Y=? Z=? (debe estar cerca del pesebre, p. ej. X=-0.5, Y=0, Z=3)
   - **Scale:** debería ser 1, 1, 1 (si es diferente, ajusta a 1, 1, 1)
   - **Rotation:** verifica que no esté girado raro (X=0, Y=0, Z=0 es lo normal)
3. Si necesitas mover/rotar: ajusta los valores directamente en los campos

**Si el modelo se ve muy grande o pequeño:**
- Modifica **Scale** en Transform (p. ej., cambia a 0.5, 0.5, 0.5 o 2, 2, 2)

### Paso 5: Crear y aplicar material Nacimiento_Toon_Blue

**¿Material NO existe aún?**

1. En **Project**, navega a `Assets/_Base/Art/` (o crea carpeta `Materials` si no está)
2. Clic derecho en el espacio vacío → **Create** → **Material**
3. Se crea "New Material" → renombra a `Nacimiento_Toon_Blue`
4. Doble-clic en `Nacimiento_Toon_Blue` para editarlo
5. En **Inspector**, busca **Shader** (arriba)
6. Clic en el dropdown "Standard" → busca y selecciona `Unlit/Color` (simple)
   - O si tienes shader toon personalizado: usa ese
7. Cambia el color:
   - **Color:** ajusta el color a azul noche oscuro
   - Clic en el cuadro de color → selecciona azul (#0A1B3F ≈ azul noche)

**Aplicar el material a NPC_Maria:**

1. Vuelve a seleccionar `NPC_Maria` en Hierarchy
2. En **Inspector**, busca **MeshRenderer**
3. En **Materials**, verás "Default-Material" o similar
4. Clic en el círculo junto a Materials[0] → busca `Nacimiento_Toon_Blue`
5. Selecciona → se aplica
6. En la **Scene view** (centro), deberías ver a Maria con color azul

### Paso 6: Repetir con NPC_Jose

1. En **Hierarchy**, busca y selecciona `NPC_Jose`
2. Repite Pasos 3-5 exactamente igual, pero usa `SM_Jose.glb` en lugar de `SM_Maria`

**Resultado esperado:**
- Dos personajes con modelo 3D en lugar de primitivas
- Color azul noche consistente

---

## FASE 3: Verificar animales

### Paso 1: Ejecutar play mode y leer consola

1. En la barra de herramientas (top), clic en **Play** (botón triangular ▶)
2. El juego comienza
3. Abre **Console** (Window → General → Console, o Tab arriba)
4. Deberías ver mensajes tipo:
   ```
   [SimInput] Contrato de input listo...
   [Director] Story01_Nacimiento: 4 beats registrados.
   [Beat] Activado: Beat1_Star
   [Beat1] Reyes Magos en camino...
   [Bootstrap01] E.3 HITO: NPC_Camello colocado (SM_Camello.glb).
   ```

### Paso 2: Verificar qué animales aparecen

En la **Scene view** (ventana central), busca:
- ✅ `SM_Camello` — debe estar visible (confirmado en consola)
- ? `SM_Burro` — ¿lo ves? (debería estar cerca del pesebre)
- ? `SM_Buey` — ¿lo ves? (debería estar cerca del pesebre)
- ? `SM_Oveja` — ¿ves 3 ovejas pequeñas? (cerca del pesebre)

**Leer consola en detalle:**
1. En Console, busca líneas que digan:
   - `[Bootstrap01] E.3 HITO: NPC_Burro colocado`
   - `[Bootstrap01] E.3 HITO: NPC_Buey colocado`
   - `[Bootstrap01] E.3 HITO: NPC_Oveja colocado`
2. Si VES estos mensajes → los animales ya se crean dinámicamente ✅
3. Si NO los ves → necesitas editar Bootstrap01.cs

### Paso 3: Si los animales NO aparecen → Editar Bootstrap01.cs

**Abre el script:**
1. En **Project**, navega a `Assets/_Base/Scripts/Story01/`
2. Doble-clic en `Bootstrap01.cs` → se abre en tu editor de código (VS Code, Rider, etc.)

**Busca la sección que crea objetos:**
- Mira alrededor de la línea ~95 donde dice algo como:
  ```csharp
  // Instanciar camello
  Instantiate(camiloPrefab, ...);
  ```

**Si hay líneas comentadas (con // al inicio):**
- Descomenta (elimina los `//`) de:
  ```csharp
  // Instantiate(burroPrefab, ...);
  // Instantiate(bueyPrefab, ...);
  // Instantiate(ovejaPrefab, ...);
  ```
- Quedaría:
  ```csharp
  Instantiate(burroPrefab, ...);
  Instantiate(bueyPrefab, ...);
  Instantiate(ovejaPrefab, ...);
  ```

**Guarda el archivo (Ctrl+S)**

**Vuelve a Unity:**
1. Click en la ventana de Unity
2. Espera a que recompile (verás "Compiling..." abajo)
3. Clic en Play nuevamente
4. Verifica en Console que ahora aparecen todos los animales

---

## FASE 4: Revisar Beat1_Star (Estrella)

### Paso 1: Verificar consola durante Beat1

**Aún en play mode**, presiona la tecla **E** (palma abierta):
- Deberías ver en **Console**:
  ```
  [Beat1] Palmaspelta detectada!
  [Beat1] Chispa creada
  ```

Luego presiona **clic izquierdo** (drag) para lanzar la chispa:
- Deberías ver:
  ```
  [Beat1] Estrella lanzada al cielo
  ```

Luego **arrastra el mouse** hacia los 3 Reyes Magos (muévete por la Scene):
- Deberías ver:
  ```
  [Beat1] Rey Mago 1 llegó!
  [Beat1] Rey Mago 2 llegó!
  [Beat1] Rey Mago 3 llegó!
  [Beat1] BEAT 1 COMPLETADO
  ```

### Paso 2: Si la Estrella NO aparece → Revisar Beat1_Star.cs

**Detén play (clic en ◼ stop)**

**Abre el script:**
1. En **Project**, navega a `Assets/_Base/Scripts/Story01/`
2. Doble-clic en `Beat1_Star.cs`

**Busca esta línea (alrededor de ~30-40):**
```csharp
starPrefab = Resources.Load<GameObject>("SM_Estrella");
```
O similar. Si dice `Resources.Load`, el modelo debe estar en `Assets/Resources/`.

**Si está aquí, verifica:**
- `SM_Estrella.glb` debe estar en `Assets/_Base/Art/Models/`
- La ruta en el código debe ser correcta

**Si necesita ajustarse:**
1. Busca dónde se crea la estrella: `Instantiate(starPrefab, ...);`
2. Verifica que `starPrefab` esté asignado en el Inspector
3. O reemplaza con:
   ```csharp
   starPrefab = AssetDatabase.LoadAssetAtPath<GameObject>("Assets/_Base/Art/Models/SM_Estrella.glb");
   ```

**Guarda y vuelve a Unity → recompila → Play**

### Paso 3: Confirmar en consola

Si todo está bien, en **Console** deberías ver:
```
[Beat1] Reyes Magos en camino... esperan una senal en el cielo.
```

Luego que completes los gestos:
```
[Beat1] BEAT 1/4 COMPLETADO
```

---

## TROUBLESHOOTING RÁPIDO

| Problema | Solución |
|---|---|
| Modelo es rosa/magenta | Material mal asignado → asigna `Nacimiento_Toon_Blue` |
| Modelo muy grande/pequeño | Ajusta Transform → Scale a 1,1,1 o similar |
| No aparece nada en Scene | Verifica que esté dentro del campo de cámara (Position Z cercano) |
| Console muestra error de prefab | Verifica ruta exacta en Project: `Assets/_Base/Art/Models/SM_*.glb` |
| Gestos no funcionan | Presiona las teclas correctas: E (palma), clic (drag), G (agarrar) |
| Consola vacía | Abre Console (Window → General → Console) |

---

## RESUMEN: Qué esperar después de Fases 2-4

**Scene view (centro):**
- Maria y Jose visibles con modelo 3D azul
- Camello, Burro, Buey, 3 Ovejas visibles
- Estrella dorada aparece al presionar E

**Console:**
- Sin errores rojos
- Todos los hitos logueados en rojo/azul
- "BEAT 1/4 COMPLETADO" al final

**Siguiente:** Beat2 (establo, animales acariciables)

---

*Generado para Unity 6.4 · Proyecto Gran Biblioteca*
