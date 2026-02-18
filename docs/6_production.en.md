# Production (MVP)

This section delimits the operational behavior of the system in its current state under a controlled single-node production environment. It defines execution assumptions, deployment constraints, resilience limits, and the absence of advanced automation, making explicit what minimum guarantees exist and what aspects remain as technical debt. It does not constitute a definitive production design nor establish formal availability commitments or SLAs.

---

## 1. Production Scope

* **Single-node** architecture.
* **1 GPU per host** is assumed.
* **High availability (HA)** and **multi-node** deployment are not contemplated.
* Horizontal scalability is out of MVP scope.
* The practical stream limit is conditioned by both resource consumption and architectural pipeline constraints.

---

## 2. Execution Architecture

### 2.1 Engine Lifecycle

* The engine runs **persistently** and does not terminate due to inactivity.
* It waits indefinitely for `run` requests.
* If **no source connects during initialization**, the system is not considered functionally deployed and remains in a waiting state.
* In that scenario, it is assumed that `runs` will be requested again externally.
* This behavior is **subject to change** in future iterations.

### 2.2 Retry Strategy

* A **fixed retry** policy is used.
* No exponential backoff exists.
* The system is considered **UP** even if only one source has connected successfully.
* This logic is one of the weakest points of the MVP and requires additional engineering for real production.

---

## 3. Deployment

### 3.1 Supported Targets

* `x86_64` with GPU.
* `arm64` (Jetson).

### 3.2 Build Process

* Image builds are performed on **development hosts**:

  * x86 host for `x86_64` images.
  * Development Jetson for `arm64` images.
* No automated CI/CD pipeline exists.

### 3.3 Distribution and Activation

* Images are transferred manually via `rsync`-based scripts.
* Deployment involves **manually stopping and restarting** services.
* An **approximate 10-minute downtime** is assumed.
* This mechanism is not desirable for the final product and will be replaced in later phases.

---

## 4. Resource Management and Known Failures

### 4.1 Resource Saturation

* CPU, GPU, or memory saturation is an expected scenario.
* The system does not implement automatic throttling or controlled degradation mechanisms.

### 4.2 Source Availability

* The system may fail if:

  * The source is unavailable during initialization.
  * The source becomes unavailable after having connected at least once.
* Behavior in these scenarios is not fully determined and forms part of the MVP technical debt.

### 4.3 Process Failure Criteria

* Formal criteria for terminating the process (fatal vs recoverable) **are not defined**.
* Additional analysis is required for:

  * GPU or RAM OOM.
  * Pipeline deadlocks.
  * Sustained degradation (frame drop, latency).

---

## 5. Memory Pipeline and Zero-Copy

* **End-to-end zero-copy is not guaranteed**.
* Main limitations:

  * Frame modification on CPU during the drawing phase (OpenCV), especially in segmentation.
  * Models requiring `float32` input.
  * Implicit copies performed by PyTorch during inference.
* On Jetson, the unified memory architecture reduces cost compared to x86, but **does not eliminate copies**.
* Conclusion: zero-copy is only possible within the limits imposed by Python + CUDA + PyTorch.

---

## 6. Metrics and Observability

Production metrics **are not fully defined** in the MVP.

This document leaves open the determination of:

* Which metrics will be collected (FPS, frame drop, latency, resource usage).
* Level of granularity (global vs per stream).
* Exposure mechanism (logs, stdout, endpoint).

---

## 7. Configuration and Security

* Configuration policy (build-time vs runtime) **is not defined**.
* Security aspects (credentials, secrets, source authentication) are **out of MVP scope**.

---

## 8. MVP Limitations

* Manual operation.
* Accepted downtime.
* No HA.
* No robust auto-recovery.
* Limited observability.

These limitations are known and considered acceptable only in the MVP context.
