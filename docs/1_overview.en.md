# Overview

This section describes the nature of the Stream Inference Engine, its position within the overall architecture, and the responsibilities it assumes in stream processing. It defines the functional boundaries of the system, its integration model with other services, configuration and state management, and behavior under failure. Finally, it contextualizes the current state as an MVP and the planned evolution paths oriented toward scalability and observability improvements.

## System Nature
- The system operates as an inference engine that runs video processing runtimes.
- It executes exclusively business-defined rules.
- It does not make autonomous decisions or apply its own business logic.
- It provides flexibility for per-client custom inference.

---

## Role Within the Architecture
- It is part of a broader microservice architecture.
- It implements three main logical modules:
  - Inference module
  - Configuration and model management module
  - Drawing module
- These modules are integrated into an orchestrator called `inference_engine`.
- The engine retrieves jobs to execute through database polling.
- The API acts as an intermediary between other services and the engine.

---

## Core Responsibilities
- Run inference on RTSP video streams.
- Apply previously defined rules over inference results.
- Draw results directly onto frames.
- Publish the resulting stream via RTSP.
- Maintain stream execution regardless of whether consumers are connected.

---

## Configuration and Model Management
- Configurations and model weights are managed by the API.
- The engine consumes only previously loaded configurations and resources.
- Enables custom inference based on per-client specific configurations.

---

## State Management and Persistence
- Persists short-term state required to execute rules across frames.
- State is used to detect history-dependent events.
- Events may result in visual drawing or future notification.
- Long-term inference history is not persisted.

---

## Exposure and Integration
- Authentication and authorization are out of scope.
- Access control is delegated to other architecture layers.
- The system exposes RTSP streams without assuming the existence of active consumers.
- Other services may consume streams or events independently.

---

## Error Handling and Run Execution
- If a source is not reachable at startup, the run is marked as failed.
- Upon connection loss, configurable retries are applied.
- If retries are exhausted, the run is marked as failed.
- Implementation errors are considered non-recoverable.
- Run state is persisted in the database.

---

## Current System State
- The system is in MVP state.
- The primary focus was validating technical and business feasibility.
- Stress tests were conducted to identify performance limits.
- The system has room for improvement with additional development.

---

## Planned Evolution
- Evolution prioritizes system scalability.
- Scalability will impact throughput and latency.
- Precise metric measurement is currently limited.
- The two-pipeline architecture affects frame traceability.
