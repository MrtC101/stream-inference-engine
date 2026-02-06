# Engineering Decisions

## 1. Objetivos técnicos y prioridades
- **Objetivos primarios**
  - Best-effort zero-copy a lo largo del pipeline.
  - Alta extensibilidad en cantidad de modelos, pesos y tareas de inferencia.
  - Mantener siempre una salida en tiempo real (real-time output).
- **Priorización**
  - Se priorizó viabilidad funcional sobre performance absoluta.
  - El sistema se concibió como un MVP orientado a validar factibilidad técnica y del modelo de negocios.
- **Objetivos descartados o postergados**
  - Zero-copy total extremo a extremo.
  - Obtención de métricas de rendimiento completamente exactas.
  - Soporte amplio de frameworks de inferencia desde el inicio.
- **Alcance inicial de frameworks**
  - Enfoque principal en YOLO.
  - Modelos específicos como anomalib fueron explícitamente postergados.

## 2. Separación en procesos y pipelines
- **Motivación principal**
  - Incompatibilidades críticas entre CUDA, librerías de procesamiento en Python, GStreamer, PyGObject y DeepStream dentro de un mismo runtime.
- **Problemas observados**
  - Caídas del proceso Python sin captura de excepciones.
  - Fallos al:
    - Inicializar pipelines de GStreamer.
    - Acceder a GstBuffer desde transforms.
    - Iniciar el debugger de Python con GStreamer activo.
    - Crear múltiples pipelines en un mismo runtime.
    - Usar simultáneamente encoder y decoder por hardware.
- **Hipótesis técnica**
  - Librerías CUDA no reentrantes.
  - Corrupción de estado interno de memoria al cargarlas múltiples veces en un mismo runtime.
- **Decisión**
  - Separación estricta en procesos independientes usando `subprocess`.
- **Topología resultante**

Inference Engine
├─ Stream Manager Bridge (IPC)
├─ Stream Manager
│ ├─ RTSP Server
│ ├─ Pipeline 1
│ └─ Workers (0..N)
└─ Stream Workers
└─ Pipeline 2 (0..N)

- Comunicación basada en IPC (stdin/stdout) y memoria compartida.
- **Separación Pipeline 1 / Pipeline 2**
- Pipeline 1: ingestión + servidor RTSP.
- Pipeline 2: procesamiento, inferencia y dibujado.
- **Motivación adicional**
- Incompatibilidad entre runtime de servidor RTSP y librerías DeepStream.
- Aislamiento del runtime evita colisiones de dependencias (incluyendo versiones previas como Savant).
- **Costos aceptados**
- Mayor latencia.
- Mayor tiempo de inicialización.
- Mayor complejidad operativa.
- **Beneficio**
- Estabilidad del sistema y ejecución consistente.

## 3. Elección de GStreamer, DeepStream y frameworks
- **Origen de la decisión**
- GStreamer fue un requisito impuesto externamente.
- **Implementación inicial descartada**
- OpenCV + NumPy en CPU: viable funcionalmente, inviable para GPU zero-copy.
- **Motivación del cambio**
- Necesidad de consumir RTSP y generar frames directamente en GPU.
- **Stack seleccionado**
- GStreamer + DeepStream para ingestión y procesamiento GPU-native.
- TensorRT como dependencia directa de DeepStream.
- **Frameworks de inferencia**
- YOLO como primer framework por:
  - Abundancia de modelos.
  - Enfoque Python-first.
  - Alta extensibilidad sin modificar el core del sistema.
- **Protocolo**
- RTSP impuesto por las fuentes externas (limitación del dominio del problema).

## 4. Gestión de memoria y frames
- **Principio rector**
- Minimizar copias de frames (CPU↔GPU y GPU↔GPU).
- **Beneficios**
- Menor consumo de RAM y VRAM.
- Mejor throughput.
- Menor frame drop rate.
- **Decisión crítica**
- Aceptar la pérdida de identidad del frame entre pipelines.
- **Justificación**
- Preservar identidad requiere:
  - Allocación adicional de memoria compartida.
  - Header de metadata persistente por frame.
  - Alto esfuerzo de desarrollo no compatible con tiempos del MVP.
- **Consecuencia actual**
- Pipeline 1 genera frames a ~25 FPS.
- Pipeline 2 consume memoria compartida a ~45 FPS, retransmitiendo el mismo frame varias veces.
- **Decisiones técnicas específicas**
1. Uso de DeepStream para generar frames directamente en GPU.
2. Uso de CuPy para capturar referencias GPU en Python.
3. Uso de PyTorch por su abstracción de tensores desacoplada del dispositivo.
4. Dibujado directo sobre frame siempre que es posible.
5. Uso de metadata + `nvdsosd` para texto y overlays en GPU.

## 5. Concurrencia, carga y frame drop
- **Límite de streams**
- No impuesto explícitamente por código.
- Determinado empíricamente por consumo de recursos.
- **Frame drop**
- Aceptado como métrica principal.
- Es la métrica más fiel dadas las limitaciones actuales.
- **Backpressure**
- Implementado mediante colas que descartan frames antes del procesamiento.
- **Métricas**
- FPS de salida suficiente para inferir frame drop rate.
- **Control de carga**
- No implementado.
- De existir, estaría basado en consumo de recursos, no en métricas de rendimiento.

## 6. API, base de datos y desacoplamiento
- **Decisión**
- Desacoplar API e Inference Engine mediante base de datos.
- **Motivación**
- Evitar sincronización de estado entre runtimes heterogéneos.
- Permitir múltiples instancias de inference engine en el futuro.
- **Estado actual**
- Base de datos local.
- Soporte futuro para escalado horizontal no implementado.

## 7. Observabilidad (postergado)
- **Estado**
- Parcialmente abordado.
- Limitado por pérdida de identidad de frame.
- **Deuda técnica**
- Métricas precisas de latencia por frame.
- Trazabilidad completa del pipeline.

## 8. Decisiones de alcance
- **Excluido del MVP**
- Control de carga avanzado.
- Métricas exhaustivas.
- Gestión completa del lifecycle.
- **Razonamiento**
- No críticas para validar viabilidad del negocio.

## 9. Portabilidad y hardware
- **Plataformas soportadas**
- x86: Tesla V100, T4.
- Jetson: JetPack 6.1.
- **Objetivo**
- Viabilidad tanto en edge como en datacenter.
- **Resultado**
- Requisito cumplido, aunque con deuda técnica pendiente.

## 10. Evolución futura
- **Decisiones condicionantes**
- Arquitectura de pipelines desacoplados con memoria compartida.
- **Componentes reemplazables**
- Frameworks de inferencia.
- Modelos y pesos.
- **Pendiente de revisión**
- Optimización de performance.
- Compatibilidad de librerías.
- Observabilidad y métricas.
