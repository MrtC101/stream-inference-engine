# Limitaciones

Esta sección explicita las restricciones técnicas observadas en el estado actual del MVP, diferenciando límites derivados de implementación, entorno y prioridades de alcance respecto de limitaciones estructurales del diseño. Define los bordes operativos actuales del sistema en términos de concurrencia, zero-copy, métricas, escalabilidad y observabilidad, estableciendo qué comportamientos son inherentes al estado presente y cuáles constituyen deuda técnica planificada para evolución futura.

---

## Límite de Concurrencia de Streams (MVP)

- El MVP soporta actualmente hasta **6 streams concurrentes** con rendimiento aceptable.
- Este límite surge de **consumo agregado de recursos** (RAM, VRAM, encoder/decoder por hardware y cómputo GPU), no de una restricción lógica del sistema.
- El límite **no está impuesto por configuración ni hardcoded**; es el punto a partir del cual el frame drop rate supera los umbrales aceptables de real-time.
- Con optimización técnica adicional, se estima que el sistema podría escalar hasta **~10 streams** sobre el mismo hardware, dependiendo de:
  - resolución,
  - FPS de entrada,
  - complejidad del modelo,
  - tipo de tarea de inferencia.
- Aun con optimización, el hardware impone un **techo práctico** que limita la escalabilidad sin aumento de recursos.

---

## Limitaciones de Zero-Copy

- No se ha logrado un esquema de **zero-copy completo** a lo largo de todo el pipeline.
- En plataformas Jetson, aunque RAM y VRAM comparten el mismo chip, el zero-copy total no es viable dentro de los límites actuales de:
  - Python,
  - CUDA,
  - PyTorch,
  - DeepStream,
  - GStreamer.
- Durante la fase de dibujado:
  - ciertas operaciones (por ejemplo, segmentación) resultan más simples de implementar en CPU usando OpenCV,
  - lo que introduce copias explícitas o implícitas de memoria.
- Algunos modelos requieren entrada en formato `float32`, lo que implica conversiones y copias adicionales.
- En la práctica, los frameworks de inferencia realizan copias internas, por lo que este comportamiento es esperado.
- Conclusión: se aplica un enfoque **best-effort zero-copy**, manteniendo los frames en GPU el mayor tiempo posible, pero aceptando copias cuando son necesarias.

---

## Limitaciones de Métricas y Observabilidad

- La arquitectura basada en **dos pipelines desacoplados mediante memoria compartida** provoca la **pérdida de identidad de frame** entre pipelines.
- Esto impide medir de forma precisa:
  - latencia end-to-end real,
  - throughput efectivo,
  - correlación frame-a-frame entre entrada y salida.
- Actualmente:
  - el **frame drop rate** se utiliza como métrica principal para evaluar comportamiento real-time,
  - otras métricas son aproximadas.
- Implementar trazabilidad completa requeriría:
  - extender el esquema de memoria compartida con headers de metadata,
  - o introducir mecanismos explícitos de sincronización entre procesos.
- Dado el alcance del MVP, esta mejora se considera **deuda técnica**.

---

## Limitaciones de Alcance del MVP

- El sistema se desarrolló como MVP orientado a **validación técnica y de negocio**, no como producto final.
- No se implementaron:
  - mecanismos de control de carga basados en recursos,
  - límites dinámicos de ejecución de runs,
  - escalado automático.
- El número de frameworks de inferencia soportados es limitado:
  - YOLO como framework principal,
  - soporte puntual para Anomalib.
- La calidad de inferencia depende exclusivamente del modelo cargado; el sistema no aplica validación semántica de resultados.
- El sistema no garantiza:
  - accesibilidad de las fuentes,
  - estabilidad de la red,
  - calidad de imagen de entrada,
  - comportamiento de los consumidores de los streams.

---

## Deuda Técnica Resuelta

- Se eliminó la **cache de frames completos en CPU del módulo de dibujado**, reduciendo el consumo excesivo de RAM observado en versiones anteriores. El módulo mantiene cache de resultados de inferencia y metadata de dibujado entre frames, pero ya no almacena copias de frames en memoria CPU.

---

## Mejoras Diferidas

Las siguientes limitaciones se consideran abordables en fases futuras de desarrollo:

- Optimización adicional del consumo de memoria RAM y VRAM.
- Mejora del lifecycle de runs y mayor resiliencia ante fallos.
- Implementación de control de carga basado en métricas de recursos.
- Mejora de métricas y trazabilidad de frames.
- Ampliación del soporte de frameworks y tipos de modelos.
