# PLAN DE INTEGRACIÓN — Historia 1: Nacimiento
## Estado actual + Próximos pasos

**Fecha:** 10 julio 2026  
**Proyecto:** Gran Biblioteca VR  
**Estado:** Graybox completo → Fase de integración de assets finales

---

## 📊 RESUMEN EJECUTIVO

| Item | Status | Detalles |
|---|---|---|
| **Modelos 3D** | ✅ 13/13 importados | Todos los .glb listos en `Assets/_Base/Art/Models/` |
| **Scripts de juego** | ✅ Completos | 5 scripts (Bootstrap01 + 4 beats) funcionando |
| **Escena graybox** | ✅ Funcional | Primitivas + estructura base listos |
| **Integración modelos** | ⏳ **EN CURSO** | Reemplazar primitivas por .glb, ajustar materiales |
| **Audio narrativo** | ⏸️ Pospuesto | Solo 1 clip de Lumi (VO_es_Lumi_saludo) importado |
| **Música + SFX** | ❌ No iniciado | Pendiente para siguiente fase |

---

## 🎯 OBJETIVO DE ESTA SESIÓN

**Reemplazar todas las primitivas graybox de Historia 1 por modelos 3D finales**, asegurar que:
- Cada modelo esté en su posición correcta
- Los materiales sean toon-style (paleta nocturna azul + dorados)
- Los colliders estén configurados para interacción
- La escena sea completamente jugable sin errores

---

## 🧩 LISTA DE MODELOS A INTEGRAR (13 total)

### Personajes estáticos (NPC)
1. `SM_Maria` — Madre, posición (pesebre)
2. `SM_Jose` — Padre, posición (pesebre)
3. `SM_ReyMago1/2/3` — Los tres reyes, generados dinámicamente en Beat1_Star

### Animales
4. `SM_Camello` — Ya instanciado dinámicamente (confirmado en consola)
5. `SM_Burro` — En establo (acariciable)
6. `SM_Buey` — En establo (ronroneo)
7. `SM_Oveja` (x3) — En establo (persiguen cámara)

### Props interactuables
8. `SM_Estrella` — Aparece en Beat1_Star (arrastrable)
9. `SM_Lampara` — Beat2_Stable (snap-zone)
10. `SM_CofreOro`, `SM_CofreIncienso`, `SM_CofreMirra` — Beat3_Gifts (puzzle 3 cofres)
11. `SM_BebeJesus` — Beat4_Cradle (gesto cuna)

---

## 📋 CHECKLIST DE INTEGRACIÓN

### Fase 1: Preparación y validación
- [ ] Asegurarse de que Unity + MCP Bridge están conectados ("Running" en Window → MCP Server)
- [ ] Abrir escena `Assets/_Base/Scenes/Story01_Nacimiento.unity`
- [ ] Revisar jerarquía actual: debería tener 11 objetos raíz (P_MainCamera, ENV_Moon, etc.)

### Fase 2: Reemplazar personajes estáticos (NPC)
- [ ] Seleccionar `NPC_Maria` en inspector
- [ ] Reemplazar su mesh/modelo por `SM_Maria.glb` (drag & drop de asset)
- [ ] Verificar posición, escala, rotación
- [ ] **Aplicar material Nacimiento_Toon_Blue** (crear si no existe)
- [ ] Repetir con `NPC_Jose` → `SM_Jose`

### Fase 3: Verificar animales (ya pueden estar instanciados dinámicamente)
- [ ] Ejecutar play mode y confirmar en consola que aparecen:
  - ✅ SM_Camello (ya logueado)
  - [ ] SM_Burro
  - [ ] SM_Buey
  - [ ] SM_Oveja (×3)
- [ ] Si no aparecen automáticamente, añadirlos como prefabs en Bootstrap01.cs

### Fase 4: Revisar Beat1_Star (Estrella)
- [ ] El script `Beat1_Star.cs` genera `PROP_Star` dinámicamente
- [ ] Verificar que use `SM_Estrella.glb` como prefab
- [ ] Confirmar en consola: "[Beat1] Estrella creada" sin errores

### Fase 5: Verificar props interactuables (Beat2-3-4)
- [ ] **Beat2_Stable:** `SM_Lampara` debe ser agarrable (grab → snap-zone)
- [ ] **Beat3_Gifts:** Los 3 cofres deben responder a gesturas:
  - Oro: girar mouse 720°
  - Incienso: soplar (B + duración)
  - Mirra: verter con precisión (pointer en franja)
- [ ] **Beat4_Cradle:** `SM_BebeJesus` debe elevarse suave con gesto cuna (ESPACIO ×3s)

### Fase 6: Materiales y lighting
- [ ] Crear/ajustar material toon "Nacimiento_Toon":
  - Color base: azul noche oscuro (#0A1B3F)
  - Rim light: dorado suave (#FFD700)
  - Emisivo: para estrella en Beat4 (verde pálido)
- [ ] Verificar que todos los modelos usen este material
- [ ] Checkpoint: ejecutar play, verificar que la paleta sea consistente

### Fase 7: Prueba de juego (QA)
- [ ] Ejecutar play y completar el flujo:
  - [ ] Beat1: Encender estrella (E), lanzar (clic), arrastrar hasta reyes (3 cápsulas llegan)
  - [ ] Beat2: Heno + lámpara + manta → snap-zones completados
  - [ ] Beat3: Abrir 3 cofres (interacciones distintas)
  - [ ] Beat4: Gesto cuna 3 s → bebé se eleva, iluminación cambia
- [ ] Consola debe mostrar: "BEAT 1/4 COMPLETADO" × 4
- [ ] Final: "HISTORIA COMPLETADA" + regreso a Hub tras 5 s
- [ ] **Criterio de éxito:** 0 errores en consola, flujo visual fluido

### Fase 8: Commit de git
```bash
git add Assets/_Base/
git commit -m "Integración de Historia 1: 13 modelos 3D + materiales toon"
```

---

## 🔧 SCRIPTS A REVISAR/AJUSTAR

Si los modelos no aparecen automáticamente, revisar estos puntos de instanciación:

**Bootstrap01.cs** (línea ~95):
```csharp
// Si esto está comentado, descomentar:
Instantiate(modelPrefab, position, rotation, parentTransform);
```

**Beat1_Star.cs** (línea ~30):
```csharp
// Crear FX_Spark y convertir a PROP_Star:
PROP_Star = Instantiate(starPrefab);
```

**Beat2_Stable.cs** (línea ~50):
```csharp
// Animales deben tener colliders y tags para interacción:
animalObject.AddComponent<CapsuleCollider>();
animalObject.tag = "Interactable";
```

---

## 📝 NOTAS IMPORTANTES

### Naming convention
Seguir el patrón:
- `SM_` = Static Mesh (malla estática)
- `SK_` = Skeletal Mesh (rigged)
- Todos en `Assets/_Base/Art/Models/`

### Colliders
Cada objeto interactuable DEBE tener:
- `CapsuleCollider` o `BoxCollider` para raycast
- Tag "Interactable" para Beat scripts

### Materiales (GDD §5.1)
- Shader: `Unlit/Color` o `Standard` con renderingMode Opaque
- Texturas: ASTC 6x6 (se configura en import settings)
- Sin normal maps pesados
- Rim-light falso vía shader personalizado (crear si no existe)

### Audio
Por ahora, mantener el 1 clip VO existente. La música y SFX entran en próxima fase.

---

## 🐛 TROUBLESHOOTING

| Problema | Solución |
|---|---|
| Modelo es rosa/magenta | Shader no encontrado → usar Unlit/Color temporalmente |
| Modelo no aparece | Verificar scale (debería ser ~1 o similar a graybox) |
| Colisión no funciona | Añadir collider en Bootstrap y verificar tag "Interactable" |
| Audio lag | Usar Compressed In Memory para clips cortos |
| Material graybox visible | Verificar que Bootstrap asigna el toon-material a todos |

---

## ⏭️ PRÓXIMOS PASOS (Después de esta fase)

1. **Fase Audio Narrativa:** Generar/grabar VO para 4 beats (ElevenLabs o grabación manual)
2. **Fase Música:** Composición original o generativa (Suno/AIVA)
3. **Fase SFX:** Efectos de sonido (coro, splash, ronroneos) vía ElevenLabs SFX
4. **Fase Optimización:** Perfilado en Quest, culling, LODs
5. **Fase DLC:** Noé, David, Jericó (roadmap GDD §3.3)

---

## 📞 REPORTAR PROGRESO

Después de cada fase, reportar:
```
✅ Fase X completada
Modelos: N/13 integrados
Errores: 0
Tiempo: ~Xm
```

---

*Generado por Claude · Proyecto Gran Biblioteca · v1.0*
