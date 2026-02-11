# Producción (MVP)

Esta sección delimita el comportamiento operativo del sistema en su estado actual bajo un entorno productivo controlado de tipo single-node. Define supuestos de ejecución, restricciones de despliegue, límites de resiliencia y ausencia de automatización avanzada, dejando explícito qué garantías mínimas existen y qué aspectos permanecen como deuda técnica. No constituye un diseño definitivo de producción ni establece compromisos formales de disponibilidad o SLA.

---

## 1. Scope de Producción

* Arquitectura **single-node**.
* Se asume **1 GPU por host**.
* No se contempla **alta disponibilidad (HA)** ni despliegue **multi-node**.
* La escalabilidad horizontal queda fuera de scope del MVP.
* El límite práctico de streams está condicionado tanto por consumo de recursos como por restricciones arquitectónicas del pipeline.

---

## 2. Arquitectura de Ejecución

### 2.1 Ciclo de vida del Engine

* El engine se ejecuta de forma **persistente** y no finaliza por inactividad.
* Permanece en espera indefinida de solicitudes de `run`.
* Si **ninguna fuente conecta durante la inicialización**, el sistema no se considera desplegado funcionalmente y queda en estado de espera.
* Se asume que, en ese escenario, los `run` serán solicitados nuevamente de forma externa.
* Este comportamiento es **susceptible de cambio** en futuras iteraciones.

### 2.2 Estrategia de Reintentos

* Se utiliza una política de **retry fijo**.
* No existe backoff exponencial.
* El sistema se considera **UP** aunque solo una fuente haya conectado correctamente.
* Esta lógica constituye uno de los puntos más débiles del MVP y requiere ingeniería adicional para producción real.

---

## 3. Despliegue

### 3.1 Targets Soportados

* `x86_64` con GPU.
* `arm64` (Jetson).

### 3.2 Proceso de Build

* El build de imágenes se realiza en **hosts de desarrollo**:

  * Host x86 para imágenes `x86_64`.
  * Jetson de desarrollo para imágenes `arm64`.
* No existe un pipeline CI/CD automatizado.

### 3.3 Distribución y Activación

* Las imágenes se transfieren manualmente mediante scripts basados en `rsync`.
* El despliegue implica **detener y reiniciar manualmente** los servicios.
* Se asume un **downtime aproximado de 10 minutos**.
* Este mecanismo no es deseable para el producto final y será reemplazado en fases posteriores.

---

## 4. Gestión de Recursos y Fallos Conocidos

### 4.1 Saturación de Recursos

* La saturación de CPU, GPU o memoria es un escenario esperado.
* El sistema no implementa mecanismos automáticos de throttling ni degradación controlada.

### 4.2 Disponibilidad de Fuentes

* El sistema puede fallar si:

  * La fuente no está disponible durante la inicialización.
  * La fuente deja de estar disponible tras haberse conectado al menos una vez.
* El comportamiento ante estos escenarios no está completamente determinado y forma parte de la deuda técnica del MVP.

### 4.3 Criterios de Fallo del Proceso

* Los criterios formales para terminar el proceso (fatal vs recoverable) **no están definidos**.
* Requiere análisis adicional para:

  * OOM en GPU o RAM.
  * Bloqueos del pipeline.
  * Degradación sostenida (frame drop, latencia).

---

## 5. Pipeline de Memoria y Zero-Copy

* No se garantiza **zero-copy end-to-end**.
* Limitaciones principales:

  * Modificación de frames en CPU durante la fase de dibujo (OpenCV), especialmente en segmentación.
  * Modelos que requieren entrada en `float32`.
  * Copias implícitas realizadas por PyTorch durante inferencia.
* En Jetson, la arquitectura de memoria unificada reduce el costo respecto a x86, pero **no elimina las copias**.
* Conclusión: zero-copy solo es posible dentro de los límites impuestos por Python + CUDA + PyTorch.

---

## 6. Métricas y Observabilidad

Las métricas de producción **no están completamente definidas** en el MVP.

Este documento deja abierto determinar:

* Qué métricas serán recolectadas (FPS, frame drop, latencia, uso de recursos).
* Nivel de granularidad (global vs por stream).
* Medio de exposición (logs, stdout, endpoint).

---

## 7. Configuración y Seguridad

* La política de configuración (build-time vs runtime) **no está definida**.
* Aspectos de seguridad (credenciales, secretos, autenticación de fuentes) quedan **fuera de scope del MVP**.

---

## 8. Limitaciones del MVP

* Operación manual.
* Downtime aceptado.
* Sin HA.
* Sin auto-recovery robusto.
* Observabilidad limitada.

Estas limitaciones son conocidas y se consideran aceptables únicamente en el contexto de MVP.
