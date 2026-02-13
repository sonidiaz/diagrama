flowchart TD
  A([Gmail Trigger: email entrante]) --> B{¿messageId ya procesado?\n(Dedupe simple)}
  B -- Sí --> STOP([STOP: ya procesado])
  B -- No --> C[Pre-procesado\n- from_email/from_domain\n- subject/date\n- messageId/threadId\n- link_gmail]

  %% =========================
  %% PAÍS + ROUTING (MVP)
  %% =========================
  C --> P1[Lookup Routing (Sheet: Routing)\n- por from_domain (o from_email)\n- devuelve país + responsable_default]
  P1 --> P2{¿Routing encontrado?}
  P2 -- Sí --> P3[Set país + responsable_default]
  P2 -- No --> P4[Fallback\n- país=DEFAULT\n- responsable=Cuentas/cola] 
  P3 --> CB
  P4 --> CB

  %% =========================
  %% CLEAN BODY (MVP)
  %% =========================
  CB[Clean body MVP (Code)\n- cortar por separadores\n- cap a N chars\n- trim] --> D{¿Es reply?\n¿Existe In-Reply-To?}

  %% =========================
  %% NUEVO TRABAJO
  %% =========================
  D -- No --> NW2[LLM Extractor\nProducto/Campaña/Pieza]
  NW2 --> NW3[Append Row (Sheet: Tareas)\n- threadId\n- país\n- responsable (default)\n- estatus inicial]
  NW3 --> H1[Append Historial simple\n(messageId, threadId, tipo=nuevo, país)]
  H1 --> N1[Notificar responsable\n(email + links)] --> END([Fin])

  %% =========================
  %% REPLY
  %% =========================
  D -- Sí --> R0{¿Hay threadId?}
  R0 -- No --> RERR[Incidencia\n- notificar Cuentas\n- log básico] --> END

  R0 -- Sí --> R1[Lookup Tarea por threadId\n(Sheet: Tareas)]
  R1 --> R2{¿Tarea encontrada?}

  R2 -- No --> R3[Incidencia\n- notificar Cuentas\n- log] --> END

  R2 -- Sí --> R4[Actualizar Tarea (patch)\n- Contexto_resumen\n- last_update_at\n- (si permitido) estatus\n- NUNCA responsable manual]
  R4 --> H2[Append Historial simple\n(messageId, threadId, tipo=reply, país)]
  H2 --> R5{¿Notificar?\n(cambios relevantes o action items)}
  R5 -- Sí --> R6[Gmail Send a Responsable\n(resumen + links)] --> END
  R5 -- No --> END
