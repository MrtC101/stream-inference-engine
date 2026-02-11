# Performance

## Objetivo del capítulo
Describir el rendimiento del sistema bajo **máxima carga aceptable** en el contexto del MVP.  
El objetivo no es exponer el procedimiento de medición ni el análisis detallado, sino **mostrar resultados mínimos de benchmark** que permitan identificar el punto a partir del cual agregar un stream adicional provoca una degradación perceptible del servicio.

Este capítulo no pretende ser un documento de benchmarking exhaustivo ni comparativo.

---

## Alcance y contexto de validez
Los resultados presentados en este documento son **válidos únicamente** bajo el siguiente entorno controlado:

- Arquitectura: x86_64
- GPU: NVIDIA Tesla V100
- CPU: 6 núcleos
- RAM: 16 GB
- VRAM: 16 GB
- CUDA: 12.6
- DeepStream: 7.1
- TensorRT: 10.3
- Modelos evaluados:
  - YOLO Small
  - YOLO Medium
  - YOLO Large
- Streams homogéneos
- Resoluciones entre 480p y 1080p
- FPS de entrada: 25–30 FPS

Cualquier variación fuera de este entorno invalida la extrapolación directa de los resultados.

---

## Definición de carga
Se entiende por **carga** la cantidad de streams procesándose en paralelo, cada uno asociado a uno o más modelos de inferencia.

El incremento de carga impacta directamente en:
- FPS efectivo
- Frame drop rate
- Latencias promedio

La degradación comienza cuando el sistema no puede sostener el procesamiento en tiempo real para todos los streams activos.

---

## Criterios de aceptación (Real-Time)
El estado del sistema se clasifica según el **frame drop rate** agregado:

- **0% – 1%**  
  Tiempo real óptimo. Máximo rendimiento aceptable.
- **1% – 5%**  
  Rendimiento aceptable para el MVP.
- **> 5%**  
  El servicio deja de considerarse real-time.

Estos umbrales son específicos del MVP y están orientados a validación de negocio, no a SLA productivos.

---

## Metodología de medición (alto nivel)
- Las métricas:
  - Excluyen warm-up y creación de pipelines
  - Se capturan en régimen estable
  - Asumen streams homogéneos
- Métricas por stream:
  - Se capturan cada **N segundos**
  - Cada valor por stream es un promedio temporal
- Métricas del sistema:
  - Se capturan cada **M segundos**
  - Representan promedios agregados de todos los streams

No se realiza detección ni eliminación de outliers. Se asume su existencia y se considera deuda técnica pendiente.

---

## Métricas utilizadas
Las métricas presentadas son agregados (promedios o máximos) de métricas por stream:

- FPS
- Frame drop rate
- Latencia de procesamiento
- Latencia end-to-end aproximada
- Uso de GPU (memoria y cómputo)
- Uso de RAM y swap

El **frame drop rate** se utiliza como métrica principal de rendimiento por ser la más representativa dadas las limitaciones actuales de trazabilidad de frames.

---

## Limitaciones de medición
- No se mide jitter
- No se mide fairness entre streams
- No se mide latencia end-to-end real del sistema completo
- No se evalúa precisión, exactitud ni calidad de inferencia
- Las métricas de pipeline 2 no se consideran confiables para evaluar rendimiento real

La pérdida de identidad de frame entre pipelines impide el cálculo de latencia end-to-end real.

---

## Resultados de benchmark (resumen)

### Configuración A
_(Placeholder – completar con resultados)_

- Número de streams:
- Modelo:
- Resolución:
- FPS de entrada:
- Frame drop rate:
- FPS promedio:
- Uso de GPU:
- Uso de RAM:

---

### Configuración B
_(Placeholder – completar con resultados)_

- Número de streams:
- Modelo:
- Resolución:
- FPS de entrada:
- Frame drop rate:
- FPS promedio:
- Uso de GPU:
- Uso de RAM:

---

### Configuración C
_(Placeholder – completar con resultados)_

- Número de streams:
- Modelo:
- Resolución:
- FPS de entrada:
- Frame drop rate:
- FPS promedio:
- Uso de GPU:
- Uso de RAM:

---

## Interpretación de resultados
Los resultados muestran que el MVP presenta un **límite práctico de streams en paralelo**, determinado principalmente por consumo de GPU.

Agregar un stream adicional con características similares a los evaluados provoca:
- Incremento no lineal del frame drop rate
- Degradación progresiva del real-time

Este límite no es una restricción teórica del hardware, sino una consecuencia del estado actual de la implementación.

---

## Implicancias y evolución esperada
- El límite de streams del MVP es una **limitación de eficiencia**, no de arquitectura.
- Con mejoras en:
  - Uso de memoria
  - Zero-copy efectivo
  - Control de carga
  - Métricas más precisas  
  Es esperable sostener las métricas del mejor caso con mayor cantidad de streams sobre el mismo hardware.

La evolución futura del sistema debería priorizar **escalabilidad horizontal por stream**, manteniendo los umbrales definidos en este documento.

---

## Fuera de alcance
Este capítulo no cubre:
- Procedimientos detallados de benchmarking
- Automatización de control de carga
- Escalabilidad multinodo
- SLA productivos
- Análisis estadístico profundo de métricas
