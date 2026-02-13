# Rendimiento

Esta sección describe el modelo de evaluación de rendimiento del sistema en el contexto del MVP. Define el entorno de validez, los criterios formales de “real-time”, las métricas utilizadas y sus limitaciones. No constituye un benchmark comparativo ni expone capacidad máxima del sistema.

---

## Alcance y contexto de validez

Las consideraciones de este documento son válidas únicamente bajo un entorno controlado equivalente al utilizado durante el desarrollo del MVP:

- Arquitectura x86_64  
- GPU NVIDIA clase datacenter  
- CPU multinúcleo  
- CUDA + TensorRT + DeepStream compatibles  
- Streams homogéneos  
- Resoluciones entre 480p y 1080p  
- FPS de entrada en rango 25–30  

Cualquier variación en hardware, drivers, modelos o configuración invalida la extrapolación directa del comportamiento descrito.

---

## Definición de carga

Se entiende por **carga** la cantidad de streams procesados en paralelo, cada uno asociado a uno o más modelos de inferencia.

El incremento de carga impacta en:

- FPS efectivo por stream  
- Frame drop rate agregado  
- Latencias promedio  
- Utilización de GPU y memoria  

El sistema presenta degradación progresiva cuando la contención de recursos impide sostener el procesamiento en tiempo real para todos los streams activos.

---

## Criterios de aceptación (Real-Time)

El estado del sistema se clasifica según el **frame drop rate agregado**:

- **0% – 1%**  
  Tiempo real óptimo dentro del alcance del MVP.
- **1% – 5%**  
  Rendimiento aceptable para validación funcional.
- **> 5%**  
  El servicio deja de considerarse real-time en el contexto del MVP.

Estos umbrales están definidos con fines de validación técnica interna y no representan SLA productivos.

El estado real-time del sistema se determina utilizando el valor agregado correspondiente a `workers_frame_drop_rate_avg`.

---

## Metodología de medición (alto nivel)

- Las métricas excluyen warm-up y creación de pipelines.
- Se capturan únicamente en régimen estable.
- Se asume homogeneidad entre streams.
- Las métricas por stream se promedian temporalmente.
- Las métricas del sistema son agregaciones periódicas de todas las métricas por stream.
- No se realiza detección ni eliminación de outliers.

El objetivo es caracterizar comportamiento general bajo carga, no realizar análisis estadístico exhaustivo.

---

## Métricas utilizadas

Las métricas utilizadas son agregados persistidos por el Engine y derivados del modelo `system_metrics`.

Se consideran:

### Worker pipelines
- `workers_fps_avg`
- `workers_frame_drop_rate_avg`
- `workers_processing_latency_avg_ms`
- `workers_e2e_latency_avg_ms`

### Factory pipelines
- `factories_fps_avg`
- `factories_e2e_latency_avg_ms`

### Recursos del sistema
- `gpu_compute_percent`
- `gpu_memory_percent`
- `ram_percent`
- `cpu_percent`

El **frame drop rate** es la métrica principal de evaluación, ya que representa de forma directa la incapacidad del sistema de sostener el flujo de frames en tiempo real.

---

## Comportamiento bajo incremento de carga

A medida que aumenta la cantidad de streams en paralelo:

- El uso de GPU tiende a crecer hasta aproximarse al punto de saturación.
- El frame drop rate aumenta de forma no lineal.
- La latencia end-to-end promedio se incrementa por contención de recursos.
- El FPS efectivo por stream puede degradarse progresivamente.

El punto de degradación no es una propiedad teórica del hardware, sino una consecuencia del estado actual de la implementación.

No se publica en este documento el límite cuantitativo de streams sostenibles.

---

## Limitaciones de medición

- No se mide jitter.
- No se mide fairness entre streams.
- No se calcula latencia end-to-end real con trazabilidad de frame completa.
- No se evalúa precisión ni calidad de inferencia.
- No se realiza análisis estadístico profundo.

La pérdida de identidad de frame entre pipelines impide el cálculo exacto de latencia end-to-end real.

---

## Implicancias y evolución esperada

El límite operativo actual del MVP está determinado principalmente por eficiencia de implementación y uso de recursos, no por restricciones estructurales de arquitectura.

Mejoras en:

- Gestión de memoria
- Reducción de copias innecesarias
- Control dinámico de carga
- Instrumentación de métricas más precisa  

deberían permitir aumentar la capacidad efectiva manteniendo los umbrales definidos de real-time.

La evolución futura prioriza escalabilidad horizontal por stream y control explícito de degradación.

---

## Fuera de alcance

Este capítulo no cubre:

- Capacidad máxima exacta del sistema
- Procedimientos detallados de benchmarking
- Comparativas con otras implementaciones
- SLA productivos
- Escalabilidad multinodo
