# PASOS MANUALES — Corrección de Código

## ✅ LO QUE CORREO:

1. **Bootstrap01.cs** — Ahora crea: camello ✅, burro ✅, buey ✅, ovejas ×3 ✅
2. **Beat1_Star.cs** — Ahora crea: sparks ✅, estrella ✅, reyes magos ×3 ✅

---

## 📋 PASOS QUE HACES TÚ (MANUAL)

### PASO 1: Abre Bootstrap01.cs
1. En Unity **Project**, navega a `Assets/_Base/Scripts/Story01/`
2. Click derecho en `Bootstrap01.cs` → **Open** (se abre en tu editor: VS Code, Rider, etc.)

### PASO 2: Selecciona TODO y reemplaza
1. **Ctrl+A** (seleccionar todo el contenido)
2. Abre el archivo `Bootstrap01_CORREGIDO.cs` (que descargaste)
3. Copia TODO su contenido (**Ctrl+C**)
4. Vuelve al editor con Bootstrap01.cs abierto
5. **Ctrl+V** (pega)
6. **Ctrl+S** (guarda)

### PASO 3: Abre Beat1_Star.cs
1. En Unity **Project**, navega a `Assets/_Base/Scripts/Story01/`
2. Click derecho en `Beat1_Star.cs` → **Open**

### PASO 4: Selecciona TODO y reemplaza
1. **Ctrl+A** (seleccionar todo)
2. Abre `Beat1_Star_CORREGIDO.cs`
3. Copia TODO su contenido (**Ctrl+C**)
4. Vuelve al editor con Beat1_Star.cs
5. **Ctrl+V** (pega)
6. **Ctrl+S** (guarda)

### PASO 5: Vuelve a Unity y espera recompilación
1. Click en la ventana de Unity (para activarla)
2. Abajo derecha, espera a que diga "**Compiling...**" y luego "**Done**"
3. Si hay errores rojos, abre **Console** (Window → General → Console)

### PASO 6: Verifica que no hay errores
- Abre **Console**
- Si solo ves logs azules (sin errores rojos), está bien ✅
- Si ves rojo, cópiame el error exacto

---

## ▶️ PASO 7: Ejecuta Play y verifica

1. **Play** (botón ▶)
2. **Console** debe mostrar:
   ```
   [Bootstrap01] Escena del Nacimiento configurada...
   [Bootstrap01] E.3 HITO: NPC_Camello colocado (SM_Camello.glb).
   [Bootstrap01] E.3 HITO: NPC_Burro colocado (SM_Burro.glb).
   [Bootstrap01] E.3 HITO: NPC_Buey colocado (SM_Buey.glb).
   [Bootstrap01] E.3 HITO: NPC_Oveja x3 colocadas (SM_Oveja.glb).
   [Bootstrap01] E.3 HITO: VO de Lumi reproduciendose (4.9 s).
   [Director] Story01_Nacimiento: 4 beats registrados.
   [Beat] Activado: Beat1_Star
   [Beat1] Reyes Magos en camino... esperan una senal en el cielo.
   ```

3. **Scene view** (ventana central): deberías ver:
   - ✅ Camello
   - ✅ Burro
   - ✅ Buey
   - ✅ 3 Ovejas (pequeñas)

---

## 🎮 PASO 8: Prueba gestos Beat1

1. En **Scene**, presiona **E** (palma abierta)
   - ✅ Debe aparecer esfera dorada pequeña (spark)

2. **Clic izquierdo + drag** (lanzar)
   - ✅ La chispa desaparece, aparece **Estrella** dorada grande

3. Mueve el mouse alrededor
   - ✅ La estrella sigue al mouse

4. Lleva la estrella cerca de la posición (0, 0, 3) — el pesebre
   - ✅ Los 3 Reyes deben aparecer y moverse hacia la estrella
   - ✅ Console muestra:
     ```
     [Beat1] Rey Mago 1 llegó!
     [Beat1] Rey Mago 2 llegó!
     [Beat1] Rey Mago 3 llegó!
     [Beat1] BEAT 1/4 COMPLETADO
     ```

---

## ✅ SI ALGO FALLA

| Síntoma | Solución |
|---|---|
| `Error: 'SimInput' does not exist` | Verifica que SimInput.cs existe en `Assets/_Base/Scripts/Core/` |
| `Error: 'AssetDatabase' not found` | Agrega `using UnityEditor;` al inicio de Bootstrap01.cs |
| Los animales no aparecen | Verifica en Project que los .glb existen: `Assets/_Base/Art/Models/SM_Burro.glb`, etc. |
| La chispa no aparece | Verifica que tienes un **Shader "Standard"** disponible en Unity |
| Los reyes no llegan | Los reyes necesitan estar a una distancia > 5m para verlo. Arrastra la estrella de lejos hacia el pesebre. |

---

## 📊 CHECKLIST FINAL

- [ ] Descargué `Bootstrap01_CORREGIDO.cs`
- [ ] Descargué `Beat1_Star_CORREGIDO.cs`
- [ ] Reemplacé Bootstrap01.cs (copiar/pegar completo)
- [ ] Reemplacé Beat1_Star.cs (copiar/pegar completo)
- [ ] Unity recompiló sin errores
- [ ] Play mode: aparecen 4 animales (camello, burro, buey, ovejas)
- [ ] Presiono E → aparece chispa dorada
- [ ] Clic+drag → chispa se convierte en estrella
- [ ] Muevo estrella → reyes aparecen y llegan
- [ ] Console muestra "BEAT 1/4 COMPLETADO"

---

## 📞 CUANDO TERMINES, REPORTA:

```
✅ Beat1 funcionando:
- Animales: Camello, Burro, Buey, Ovejas ×3 ✅
- Spark → Estrella ✅
- Reyes Magos ×3 llegan ✅
- Console sin errores ✅
```

O si hay error:
```
❌ Error: [pega el error exacto de la consola]
```

---

*Generado para Unity 6.4 · Proyecto Gran Biblioteca*
