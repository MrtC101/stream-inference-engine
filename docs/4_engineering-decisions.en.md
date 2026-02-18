# Engineering Decisions

This chapter documents the structural technical decisions that defined the current system architecture, including motivations, environment constraints, discarded alternatives, and accepted costs. It exposes how the MVP priorities—stability, functional viability, and extensibility—conditioned choices in process separation, GPU usage, memory management, metrics, and decoupling between components, as well as the technical implications these decisions generate for the future evolution of the engine.

## 1. Technical Goals and Priorities
- **Primary objectives**
  - Best-effort zero-copy throughout the pipeline.
  - High extensibility in the number of models, weights, and inference tasks.
  - Always maintain a real-time output.
- **Prioritization**
  - Functional viability was prioritized over absolute performance.
  - The system was conceived as an MVP aimed at validating technical and business feasibility.
- **Discarded or deferred objectives**
  - Full end-to-end zero-copy.
  - Completely accurate performance metrics.
  - Broad inference framework support from the start.
- **Initial framework scope**
  - Primary focus on YOLO.
  - Specific models such as Anomalib were explicitly deferred.

## 2. Process and Pipeline Separation
- **Primary motivation**
  - Critical incompatibilities between CUDA, Python processing libraries, GStreamer, PyGObject, and DeepStream within the same runtime.
- **Observed problems**
  - Python process crashes without exception capture.
  - Failures when:
    - Initializing GStreamer pipelines.
    - Accessing GstBuffer from transforms.
    - Starting the Python debugger with GStreamer active.
    - Creating multiple pipelines in the same runtime.
    - Using hardware encoder and decoder simultaneously.
- **Technical hypothesis**
  - Non-reentrant CUDA libraries.
  - Internal memory state corruption when loaded multiple times in the same runtime.
- **Decision**
  - Strict separation into independent processes using `subprocess`.
- **Resulting topology**

```
Inference Engine
├─ Stream Manager Bridge (IPC)
├─ Stream Manager
│  ├─ RTSP Server
│  ├─ Factory Pipeline
│  └─ Workers (0..N)
└─ Stream Workers
   └─ Inference Pipeline (0..N)
```

- Communication based on IPC (stdin/stdout) and shared memory.
- **Factory Pipeline / Inference Pipeline separation**
- Factory Pipeline (Stream Manager): reads from shared memory (shmsrc) and feeds the RTSP server.
- Inference Pipeline (Workers): RTSP ingestion → inference → drawing → H264 encode → shared memory (shmsink).
- **Additional motivation**
- Incompatibility between RTSP server runtime and DeepStream libraries.
- Runtime isolation avoids dependency collisions (including previous versions such as Savant).
- **Accepted costs**
- Higher latency.
- Longer initialization time.
- Greater operational complexity.
- **Benefit**
- System stability and consistent execution.

## 3. Choice of GStreamer, DeepStream, and Frameworks
- **Origin of the decision**
- GStreamer was an externally imposed requirement.
- **Initial implementation discarded**
- OpenCV + NumPy on CPU: functionally viable, not viable for GPU zero-copy.
- **Motivation for the change**
- Need to consume RTSP and generate frames directly on GPU.
- **Selected stack**
- GStreamer + DeepStream for GPU-native ingestion and processing.
- TensorRT as part of the NVIDIA runtime (available through the DeepStream SDK and CUDA Toolkit, not as a declared project dependency).
- **Inference frameworks**
- YOLO as the primary framework due to:
  - Abundance of models.
  - Python-first approach.
  - High extensibility without modifying the system core.
- **Protocol**
- RTSP imposed by external sources (domain problem constraint).

## 4. Memory and Frame Management
- **Guiding principle**
- Minimize frame copies (CPU↔GPU and GPU↔GPU).
- **Benefits**
- Lower RAM and VRAM consumption.
- Better throughput.
- Lower frame drop rate.
- **Critical decision**
- Accept loss of frame identity between pipelines.
- **Justification**
- Preserving identity requires:
  - Additional shared memory allocation.
  - Persistent metadata header per frame.
  - High development effort incompatible with MVP timelines.
- **Current consequence**
- The Inference Pipeline generates processed frames at ~25 FPS in shared memory.
- The Factory Pipeline consumes shared memory at ~45 FPS, retransmitting the same frame multiple times to the RTSP server.
- **Specific technical decisions**
1. Use of DeepStream to generate frames directly on GPU.
2. Use of CuPy to capture GPU references in Python.
3. Use of PyTorch for its device-decoupled tensor abstraction.
4. Direct drawing on frame whenever possible.
5. Use of metadata + `nvdsosd` for text and overlays on GPU.

## 5. Concurrency, Load, and Frame Drop
- **Stream limit**
- Not explicitly imposed by code.
- Determined empirically by resource consumption.
- **Frame drop**
- Accepted as the primary metric.
- It is the most faithful metric given current limitations.
- **Backpressure**
- Implemented through queues that discard frames before processing.
- **Metrics**
- Output FPS sufficient to infer frame drop rate.
- **Load control**
- Not implemented.
- If implemented, it would be based on resource consumption, not performance metrics.

## 6. API, Database, and Decoupling
- **Decision**
- Decouple API and Inference Engine through the database.
- **Motivation**
- Avoid state synchronization between heterogeneous runtimes.
- Allow multiple inference engine instances in the future.
- **Current state**
- Local database.
- Future support for horizontal scaling not implemented.

## 7. Observability (Deferred)
- **State**
- Partially addressed.
- Limited by loss of frame identity.
- **Technical debt**
- Precise per-frame latency metrics.
- Full pipeline traceability.

## 8. Scope Decisions
- **Excluded from MVP**
- Advanced load control.
- Exhaustive metrics.
- Full lifecycle management.
- **Reasoning**
- Not critical for validating business feasibility.

## 9. Portability and Hardware
- **Supported platforms**
- x86: Tesla V100, T4.
- Jetson: JetPack 6.1.
- **Objective**
- Viability in both edge and datacenter environments.
- **Result**
- Requirement met, though with pending technical debt.

## 10. Future Evolution
- **Conditioning decisions**
- Decoupled pipeline architecture with shared memory.
- **Replaceable components**
- Inference frameworks.
- Models and weights.
- **Pending review**
- Performance optimization.
- Library compatibility.
- Observability and metrics.
