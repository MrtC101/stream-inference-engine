# Motor de Inferencia de streams

Este documento define el alcance funcional y operativo del Stream Inference Engine. Describe el problema que resuelve, las restricciones impuestas por el contexto del MVP, los supuestos técnicos bajo los que opera y los límites explícitos de responsabilidad del sistema. No detalla implementación interna ni decisiones de ingeniería específicas, sino el marco contractual y operativo bajo el cual el sistema debe evaluarse.

## Propósito del sistema (Problem Space)

- Procesar múltiples streams de video concurrentes en tiempo real.
- Ejecutar inferencia sobre streams de video y generar salida visual.
- Publicar el resultado como stream de video, no como metadata estructurada.
- Operar en contexto productivo B2B, no como servicio genérico o self-service.
- Definir “tiempo real” mediante frame-drop rate (< 5% respecto a la entrada).
- Se considera tiempo real cuando el frame-drop rate < 5% respecto al framerate de entrada.

---

## Supuestos del contexto (Assumptions)

- Las fuentes de video son cámaras remotas externas.
- Las fuentes exponen streams vía RTSP.
- El codec de entrada es H.264 (impuesto por el contexto del MVP).
- El protocolo de salida es RTSP.
- La inferencia se ejecuta prioritariamente sobre GPU.
- El sistema se despliega sobre plataformas x86_64 y arm64 (Jetson).
- Varias decisiones técnicas fueron impuestas por el contexto organizacional del MVP.

---

## Restricciones no negociables (Constraints)

- El sistema no controla el protocolo ni el codec de las fuentes.
- El sistema no controla la disponibilidad ni estabilidad de la red.
- La salida debe mantenerse en tiempo real bajo la métrica definida.
- La salida es exclusivamente video con overlays; no se emite metadata adicional.
- Se priorizan continuidad, estabilidad y latencia por sobre optimización extrema.
- El servicio se entrega directamente en un contexto B2B (sin elasticidad automática).

---

## Postura operativa del sistema

- El sistema valida la alcanzabilidad de la fuente antes de iniciar el procesamiento.
- Ante pérdida de conexión, el sistema reintenta según configuración.
- Si se excede el número de reintentos, el stream se considera fallido.
- En el estado actual del MVP no existen mecanismos automáticos de limitación de carga.
- Solo se ofrecen configuraciones consideradas aceptables para no degradar performance.

---

## Límites explícitos de responsabilidad (Responsibility Boundaries)

- El sistema no garantiza la calidad ni corrección de la inferencia.
- La accuracy es responsabilidad del modelo cargado.
- El sistema no garantiza accesibilidad de las fuentes.
- Fallas de red están fuera de scope.
- Fallas del emisor del stream están fuera de scope.
- Fallas de sistemas consumidores de la salida están fuera de scope.
- El sistema no garantiza optimización óptima de recursos en el estado actual del MVP.

---

## Estado actual del sistema

- El sistema se encuentra en estado MVP.
- Demuestra viabilidad funcional end-to-end:
  - inicio de streams
  - ejecución de inferencia
  - visualización del resultado en un frontend
- Existen limitaciones conocidas de consumo de recursos (RAM, uso parcial de NVENC/NVDEC).
- Estas limitaciones se reconocen como deuda técnica y no como fallas funcionales.
