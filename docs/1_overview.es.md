# Stream inference engine

# Overview

## Naturaleza del sistema
- El sistema funciona como un motor de inferencia que ejecuta runtimes de procesamiento de video.
- Ejecuta exclusivamente reglas definidas por el negocio.
- No toma decisiones autónomas ni aplica lógica de negocio propia.
- Brinda flexibilidad para inferencia personalizada por cliente.

---

## Rol dentro de la arquitectura
- Es parte de una arquitectura de microservicios más amplia.
- Implementa tres módulos lógicos principales:
  - Módulo de inferencia
  - Módulo de configuraciones y modelos
  - Módulo de dibujado
- Estos módulos se integran en un orquestador denominado `inference_engine`.
- El engine obtiene los trabajos a ejecutar mediante polling en una base de datos.
- La API actúa como intermediario entre otros servicios y el engine.

---

## Responsabilidades principales
- Ejecutar inferencia sobre streams de video RTSP.
- Aplicar reglas previamente definidas sobre los resultados de inferencia.
- Dibujar los resultados directamente sobre los frames.
- Publicar el stream resultante mediante RTSP.
- Mantener la ejecución de los streams independientemente de la existencia de consumidores.

---

## Gestión de configuraciones y modelos
- Las configuraciones y pesos de modelos son administrados por la API.
- El engine consume únicamente configuraciones y recursos previamente cargados.
- Permite inferencia personalizada a partir de configuraciones específicas por cliente.

---

## Manejo de estado y persistencia
- Persiste estado de corto plazo necesario para ejecutar reglas entre frames.
- El estado se utiliza para detectar eventos dependientes de historial.
- Los eventos pueden derivar en dibujado visual o notificación futura.
- No se persiste información histórica de inferencia a largo plazo.

---

## Exposición e integración
- La autenticación y autorización están fuera de scope.
- El acceso está delegado a otras capas de la arquitectura.
- El sistema expone streams RTSP sin asumir la existencia de consumidores activos.
- Otros servicios pueden consumir streams o eventos de forma independiente.

---

## Manejo de errores y ejecución de runs
- Si una fuente no es alcanzable inicialmente, el run se marca como fallido.
- Ante pérdida de conexión, se aplican reintentos configurables.
- Si los reintentos fallan, el run se marca como fallido.
- Los errores de implementación se consideran no recuperables.
- El estado de los runs se persiste en la base de datos.

---

## Estado actual del sistema
- El sistema se encuentra en estado MVP.
- El foco principal fue la validación de viabilidad técnica y de negocio.
- Se realizaron pruebas de estrés para identificar límites de rendimiento.
- El sistema presenta margen de mejora con desarrollo adicional.

---

## Evolución prevista
- La evolución prioriza escalabilidad del sistema.
- La escalabilidad impactará en throughput y latencia.
- La medición precisa de métricas es actualmente limitada.
- La arquitectura de dos pipelines afecta la trazabilidad de los frames.
s