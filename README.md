# Pre-entrega 1 · Agente autónomo básico funcional (n8n)

**Autora:** Deidy Perez · AI Automation Avanzado (Coderhouse, 2026)
**Archivo entregado:** `checkpoint1_deidy_perez.json`

## Caso de negocio
Asistente de Soporte a Vendedores Particulares de un marketplace automotor chileno de vehículos usados. Responde por qué un aviso no aparece, si fue rechazado, si el pago está pendiente, cuándo vence y cuántos contactos recibió.

## Arquitectura
```
[Chat Trigger] → [AI Agent · Tools Agent · Gemini · máx. 8 iteraciones]
                        └─ Tool lateral: consultar_estado_aviso (Airtable · Search)
               → [Slack · Log de observabilidad "Tarea completada"] → [Respuesta al chat]
```

| Requisito | Cómo se cumple |
|---|---|
| Trigger | `Chat Trigger` recibe el mensaje libre del vendedor |
| AI Agent en Tools Agent | Nodo `Agente de Soporte a Vendedores`, Agent = **Tools Agent** |
| Modelo | `Google Gemini Chat Model` (gemini-3.1-flash-lite, temperatura 0.2) |
| Guardrail de iteraciones | Options → **Max Iterations = 8** |
| System Prompt modular | Rol → Ámbito → Objetivo → Herramientas → Reglas → Escalamiento; prohíbe modificar avisos, pagos o precios |
| Tool lateral con descripción extensa | `consultar_estado_aviso` (Airtable) conectada al puerto *Tool* del agente, con descripción de cuándo usarla y cuándo no |
| Observabilidad | Slack publica: mensaje, herramientas usadas, iteraciones, resultado de la herramienta, respuesta y si requiere escalamiento |

## Cómo importarlo
1. n8n → *Import from File* → `checkpoint1_deidy_perez.json`.
2. Credenciales: Google Gemini (PaLM) API, Airtable OAuth2 (solo lectura sobre la base) y Slack OAuth2 (canal `#soporte-ia-logs`).
3. En `consultar_estado_aviso` elegir la base y la tabla `Avisos` (columnas: ID_Aviso, Patente, Email_Vendedor, Vehiculo, Precio_CLP, Estado_Aviso, Motivo_Estado, Paquete, Fecha_Publicacion, Fecha_Vencimiento, Leads_Recibidos).

## Prueba de validación
- `¿Por qué rechazaron mi aviso? Mi patente es GZDF60` → el agente usa la tool y explica el motivo.
- `¿Cómo saco buenas fotos para mi auto?` → responde sin usar la tool.
- `Me cobraron dos veces` → deriva a un ejecutivo (escalamiento).

> No se incluyen API keys: las credenciales viven en n8n y no se exportan en el JSON.
