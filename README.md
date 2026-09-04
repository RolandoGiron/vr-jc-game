# vr-jc-game

Juego VR de historias bíblicas en Unity. **Este repo contiene solo la documentación**
(GDD, guías de fase, correcciones de playtest, pipeline de rig). El proyecto Unity vive
aparte, en `C:\Users\rolan\test_mcp_unity`.

## Documentos

| Archivo | Qué es |
|---|---|
| `GDD.md` | Documento de diseño del juego |
| `guia_fase1_historias_base.md` · `fase1_cierre_validacion.md` | Fase 1: historias base y su validación |
| `GUIA_DETALLADA_FASES_2_3_4.md` | Fases 2, 3 y 4 |
| `PLAN_INTEGRACION_HISTORIA1.md` | Integración de la Historia 1 (Nacimiento) |
| `CORRECCIONES_PLAYTEST_AGO2026.md` | Las cuatro rondas de playtest de agosto 2026 y cómo se corrigió cada hallazgo |
| `RIG_PERSONAJES.md` | Auto-rig de los modelos Meshy con Blender headless |
| `guia_poc_unity_mcp.md` · `PASOS_MANUALES_CORRECCION.md` | Notas del PoC inicial (históricas) |

## Convenciones del proyecto Unity

Reglas que ya costaron un bug, útiles antes de tocar código:

- **Es URP.** `Shader.Find("Standard")` devuelve null y pinta magenta. Usar
  `GrayboxFactory.Mat(color)` o `FXKit.Emissive(color, intensidad)`.
- **Todo se construye en runtime por código**; las escenas están casi vacías.
  `PolishDirector` se autoinstala en cada escena y rescanea cada 0.5 s.
- **`ArtDresser` viste por nombre de objeto.** Si un beat instancia el modelo él mismo,
  hay que colgarlo como hijo `<NombreHost>_Visual`, o en el siguiente rescaneo aparece un
  segundo modelo dentro del primero.

## Cómo levantar el MCP de Unity

Desde septiembre de 2026 conviven dos servidores MCP. La carpeta `unity-mcp-master` de
este repo corresponde al paquete antiguo (`com.justinpbarnett.unity-mcp`, abandonado
en 2025) y **ya no se usa**.

### MCP for Unity (CoplayDev) — el que edita el proyecto

Paquete `com.coplaydev.unity-mcp` v10.0.0, declarado por git URL en
`Packages/manifest.json`. Requiere `uv` y Python 3.10+.

1. En Unity: `Window > MCP for Unity` (Ctrl+Shift+M), pestaña **Connect**.
2. **Transport: `Stdio`** — no "HTTP Local". Claude Desktop lanza el servidor por stdio;
   con HTTP el panel muestra "Session Active" pero es otra sesión y todas las tools
   responden `No Unity Editor instances found`. El aviso "Transport Mismatch" es
   exactamente ese fallo, no un adorno.
3. **Start Session**. En consola debe salir `StdioBridgeHost started on port 6400`.

Tras cada recompilación de scripts la sesión se cae a "No Session": hay que volver a
pulsar Start Session.

### MCP oficial de Unity — inspección

Paquete `com.unity.ai.assistant` (Unity 6.4). Panel en
`Edit > Project Settings > AI > Unity MCP Server`. El bridge arranca solo al abrir el
editor y no se cae al recompilar, así que sirve de respaldo para inspeccionar jerarquía,
consola, profiler y capturas mientras el otro está desconectado.
